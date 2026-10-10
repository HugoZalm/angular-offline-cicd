# Offline CI-CD stack for an angular dev-environment

## The Problem
I have a laptop without internet.
But i want a CI/CD stack for my angular frontend project.

So i want a docker compose stack - wich i can run on any computer - for my CI/CD stack.
The docker compose part must be transpossable to other projects. The project content must be stored in mounted volumes, so they stay on my computer (or can be shipped to my teammebers by USB-stick).

I also want to use free and opensource programmes, so i don't have to bother about licenses.

In my stack there should be:

* An angular development environment
* A repo-system (prefferably git)
* A ticket system, so i can track my progress
* A system to build and publish my work to an OTAP street (DEV-TEST-ACC-PROD environments)

The stack should be able to talk to other docker compose stacks on my computer: the API (with database etc) will be created by other teammembers and will run in a different environment in docker on my laptop.

## My solution so far

### The offline catch

Offline, nothing can be downloaded: no Docker images, no npm packages. So i need a **preparation phase while online** (once, or on a colleague's machine), then everything runs offline:

1. **Docker images:** pull them once, then `docker save -o images.tar <image1> <image2> ...`. On the offline laptop, run `docker load -i images.tar`. This tarball goes on the USB stick.
2. **npm packages:** run a local npm registry (Verdaccio) that caches packages. While online, run `npm ci` once through it. Offline, it serves everything from its cache, which lives in a mounted volume.

### Recommended stack (all open source)

| Need | Tool | Why |
|---|---|---|
| Git + tickets | **Gitea** | Light (about 200 MB RAM). Git, issues, labels, milestones and kanban boards. |
| CI/CD | **Gitea Actions + act_runner** | Built into Gitea, GitHub Actions-style YAML. |
| Angular dev environment | **node container with Angular CLI** (optionally **code-server**, VS Code in the browser) | Source stays in a mounted folder. |
| npm offline cache | **Verdaccio** | See above. |
| Image registry | **registry:2** | Stores built images for promotion between environments. |
| OTAP | One small compose project per environment, **nginx** serving the Angular build | Same image in every environment. Only the config differs. |

GitLab CE would be an all-in-one alternative, but it needs 4 GB+ RAM and is heavy for a laptop. Gitea is the better fit here.

### Project layout

```
cicd-stack/                 # reusable for any project
├── docker-compose.yml
├── .env
└── data/                   # everything persistent lives here (USB-able)
    ├── gitea/  runner/  verdaccio/  registry/
workspace/                  # my Angular project (git clone)
```

### One-time setup

```bash
docker network create shared-net     # all stacks (mine + the API team's) join this
```

### `.env`

```env
SHARED_NETWORK=shared-net
RUNNER_TOKEN=            # filled in after Gitea is up (step 2 below)
```

### `docker-compose.yml`

```yaml
services:
  gitea:
    image: gitea/gitea:1.22
    restart: unless-stopped
    environment:
      GITEA__server__ROOT_URL: http://localhost:3000/
      GITEA__actions__ENABLED: "true"
      GITEA__database__DB_TYPE: sqlite3
    volumes:
      - ./data/gitea:/data
    ports:
      - "3000:3000"
      - "2222:22"
    networks: [cicd, shared]

  runner:
    image: gitea/act_runner:latest
    restart: unless-stopped
    depends_on: [gitea]
    environment:
      GITEA_INSTANCE_URL: http://gitea:3000
      GITEA_RUNNER_REGISTRATION_TOKEN: ${RUNNER_TOKEN}
      GITEA_RUNNER_NAME: local-runner
      # job containers use LOCAL images (no internet needed)
      GITEA_RUNNER_LABELS: "node20:docker://node:20-bookworm,docker:docker://docker:27-cli"
    volumes:
      - ./data/runner:/data
      - /var/run/docker.sock:/var/run/docker.sock   # lets jobs build/deploy containers
    networks: [cicd, shared]

  verdaccio:
    image: verdaccio/verdaccio:5
    restart: unless-stopped
    volumes:
      - ./data/verdaccio:/verdaccio/storage
    ports:
      - "4873:4873"
    networks: [cicd]

  registry:
    image: registry:2
    restart: unless-stopped
    volumes:
      - ./data/registry:/var/lib/registry
    ports:
      - "5000:5000"
    networks: [cicd]

  dev:                       # Angular dev environment
    image: node:20-bookworm
    working_dir: /workspace
    environment:
      npm_config_registry: http://verdaccio:4873
    volumes:
      - ./workspace:/workspace
    command: sh -c "npm ci && npx ng serve --host 0.0.0.0 --poll 2000"
    ports:
      - "4200:4200"
    networks: [cicd, shared]

networks:
  cicd:
  shared:
    external: true
    name: ${SHARED_NETWORK}
```

Start with `docker compose up -d gitea verdaccio registry`. Then:

1. Open http://localhost:3000, finish the install wizard, and create your account and repo.
2. Go to *Site Administration → Actions → Runners → Create new runner* and copy the token into `.env`.
3. Run `docker compose up -d runner dev`.

For tickets, enable *Issues* and *Projects* in the repo settings to get a kanban board.

### Talking to the API stack

The API team's compose file should join the same network:

```yaml
networks:
  default:
    name: shared-net
    external: true
```

Then the Angular nginx (or the `ng serve` proxy) can reach it by service name, e.g. `http://api:8080`. This works as long as both stacks are on `shared-net`. Agree on the service names with the team.

### OTAP: build once, promote the same image

- Build the Angular app **once** into an nginx image and push it to `localhost:5000/myapp:<git-sha>`.
- Don't bake environment-specific URLs into the build. Let nginx substitute them at startup (e.g. `API_URL` via an env var in the nginx template). That way the same image runs in every environment.
- Each environment is just the same small compose file run with a different project name, port and env vars:

```yaml
# deploy/compose.yml (lives in the project repo)
services:
  web:
    image: localhost:5000/myapp:${TAG}
    environment:
      API_URL: ${API_URL}
    ports: ["${PORT}:80"]
    networks: [shared]
networks:
  shared: { external: true, name: shared-net }
```

| Env | Port | Trigger |
|---|---|---|
| DEV | 8081 | every push to `develop` |
| TEST | 8082 | merge into `main` |
| ACC | 8083 | git tag `v*` |
| PROD | 8080 | manual approval (`workflow_dispatch`) |

Deploy step in the workflow:

```bash
TAG=$SHA PORT=8081 API_URL=http://api-dev:8080 \
  docker compose -p myapp-dev -f deploy/compose.yml up -d
```

### Making it reusable for other projects

The stack above is project-independent. For a new project we only need to:

- create a new repo in Gitea,
- clone it into `./workspace`,
- copy the `.gitea/workflows/` and `deploy/` folders and change the app name and ports.

### Practical notes

- **Docker socket:** mounting it gives jobs root-level control of Docker. That's fine on my own laptop, but we must be aware of it and find out the consequences.
- **Runner images:** the `node:20-bookworm` and `docker:27-cli` images must be in your `docker save` tarball, or CI jobs can't start offline.
- **Windows/macOS:** `ng serve --poll` is needed so file changes in mounted volumes are detected.
- **Git over SSH:** works on port 2222. HTTP on port 3000 is simpler.

## Some general notes

### Versions
- use angular 22 (latest at this time)
- i suppose docker should be installed locally on my PC. Wha tversion do i need ? And will it make a difference if i and my team-members use different versions (or WSL yes/no, linux/windows etc)

### When online
- What happens if we are online: can we use the same stack with bennifits ? (use AI like opencode or claude code) ?
