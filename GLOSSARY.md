# Offline CI/CD Stack

A reusable, project-independent Docker Compose stack that gives an Angular developer
a full CI/CD environment — Git hosting, tickets, a runner, an npm cache, an image
registry and DTAP environments — on a machine with no internet, with all state living
in USB-shippable mounted volumes.

## Language

### The platform

**Stack**:
The reusable, project-independent Compose environment (`stack/`) that hosts many
projects over time: Gitea, the runner, Verdaccio, the registry and the dev container.
_Avoid_: platform, infra, server

**Project**:
A user's own Angular repository that consumes the stack by applying the
copy-the-files contract.
_Avoid_: app, workspace

**Runner**:
The `act_runner` service that registers with Gitea and executes jobs as local
containers, using the pinned Node 22 LTS label and the mounted Docker socket.
_Avoid_: agent, worker, executor

**Gitea Issues / Board**:
The product's ticketing feature — issues, labels, milestones and the Projects kanban
board — configured inside Gitea. This is distinct from the repo's own build
coordination (see note below).
_Avoid_: issue tracker, ticketing system

**Shared network**:
The external Docker network (`SHARED_NETWORK`, default `shared-net`) that lets the
stack and other Compose stacks on the machine — such as the API team's — reach each
other by service name.
_Avoid_: bridge, overlay

### Offline preparation

**Preparation (Prep)**:
The one-time online phase that makes an offline laptop possible: pulling and saving
images, warming the npm cache and producing the artifact set. It is the only step that
touches the internet.
_Avoid_: setup, provisioning, install

**Artifact set**:
The USB payload produced by prep: the saved images, the warmed Verdaccio storage, the
manifest, the git bundles and a version stamp.
_Avoid_: bundle, payload

**Manifest**:
The machine-readable record of the artifact set — images with their tags/digests,
checksums and versions — whose diff between prep runs shows the incremental delta.
_Avoid_: index, lockfile

### Environments and delivery

**Environment**:
One deployment target the app is promoted through, defined per project in `stack.env`.
_Avoid_: stage, tier

**DTAP**:
The default environment set — Development, Test, Acceptance, Production (English for
the Dutch "OTAP"). A project may define any environment set instead.
_Avoid_: OTAP

**Promotion**:
Moving one already-built image tag unchanged from environment to environment, injecting
only runtime config (ports, `API_URL`) at startup.
_Avoid_: deployment, release, rollout

### Reuse

**Copy-the-files contract**:
The three things a project copies into its own repo to adopt the stack: `stack.env`,
`deploy/` and `.gitea/workflows/` — no shared parent repo, no submodule, no scaffolder.
_Avoid_: scaffolder, template, submodule

---

_Note: "issue tracker" is reserved for this repo's own build coordination (GitHub
Issues); the product feature is "Gitea Issues" / "the board"._
