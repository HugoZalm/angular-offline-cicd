# Spec: Offline CI/CD stack for an Angular dev environment

Status: agreed (from a `grill-with-docs` session). Origin: `brief.md`.
This document is the shared understanding reached during grilling; no project
building has started.

## Problem

A developer has a laptop without internet and wants a portable CI/CD stack for an
Angular frontend project. The compose stack must be reusable across projects, all
state must live in mounted volumes (USB-shippable), and everything must be free and
open source. It must also be able to talk to another team's compose stack (the API +
database) running on the same machine.

## Decisions (settled during grilling)

- **Q1 — Deliverable:** a reusable, project-independent stack **plus a disposable
  Angular 22 sample app** used only to prove the offline pipeline end to end. The app
  is a test fixture, not a deliverable.
- **Q2 — Support target:** Windows + Docker Desktop (WSL2 backend) is the hard
  requirement; compose files stay OS-agnostic. One Docker Desktop version is pinned as
  the tested baseline.
- **Q3 — Vocabulary:** reserve **"issue tracker"** for this repo's own build
  coordination; the product feature is **"Gitea Issues" / "the board"**. Configuring
  Gitea Issues + Projects is in scope; building a custom ticketing app is not.
- **Q4 — Offline mechanism:** `docker save`/`load` (manifest-driven scripts) as the
  default image path; **`skopeo`** documented as the multi-arch / registry-shaped
  upgrade. npm via **Verdaccio with uplink-removal + `verdaccio-offline-storage`**.
- **Q5 — Environment model:** generic; any environment set is configurable. **DTAP**
  (Development, Test, Acceptance, Production) — English for the Dutch "OTAP" — ships
  as the default.
- **Q6 — Reuse mechanism:** the **copy-the-files contract** (no shared platform repo,
  no submodule, no scaffolder).
- **Q7 — Environment config:** one declarative file as source of truth (DTAP default);
  **any project can define its own environment set**, and it must be trivial to do.
- **Q8 — The contract:** exactly **`stack.env` + `deploy/` + `.gitea/workflows/`**,
  nothing more.
- **Q9 — Repo layout:** `stack/`, `sample-app/`, `docs/`, `.scratch/`, git-ignored
  `artifacts/`.
- **Q10 — Version policy:** pin the current **stable** tuple (Docker Desktop, Compose
  v2, Gitea, Node LTS) at build time and record it; don't chase minimums. Cross-OS
  reduces to two footnotes: Linux runs Docker Engine + Compose plugin (no Desktop), and
  `--poll` is needed for mounted-volume file-watching on Windows/macOS.
- **Q11 — Modes:** there is **no online mode of the stack**. Online is *preparation
  only*; the running stack is always offline. Offline AI is out of scope for now.
- **Q12 — Prep workflow:** scripted, idempotent commands (an online-side `prep` that
  pulls/saves images and warms Verdaccio; an offline `load` + `up`). Prep is the only
  thing that touches the internet.
- **Q13 — USB payload:** `artifacts/` = `images.tar`, Verdaccio `storage/`, a
  `MANIFEST` (images + tags/digests + checksums), plus git bundles of the stack and the
  project, with a version stamp. The **stack repo lives on a real public GitHub remote**
  and contains **no secrets and no real project code**; during prep it is pulled from
  GitHub, then everything stays on the USB.
- **Q14 — Verification:** an automated **offline smoke test** that runs the whole
  pipeline with outbound network blocked and asserts a deploy lands. That test is the
  definition of done for "offline works."
- **Q15 — Adding deps offline:** impossible by design. Refresh = go online and re-run
  `prep`, made incremental (only missing images/versions; `MANIFEST` diff shows the
  delta). **Manual offline escape hatches** are allowed: hand-drop a `docker load`-able
  image tar and hand-place Angular asset files, so the offline environment can grow
  without going online.
- **Q16 — Docker socket:** accepted deliberately in `act_runner` (jobs must build and
  deploy containers). The security caveat and its rules are recorded as an ADR and
  repeated in the documentation.
- **Q17 — Where environments live:** in the **project's `stack.env`** (per project);
  the shared stack only provides the generic mechanism.
- **Q18 — Tenancy:** one shared stack instance per machine, hosting many projects over
  time. "Reuse" = add a repo + apply the contract. This is what `shared-net` is for.
- **Q19 — This repo's own tracker:** GitHub Issues (the skills' native home).
- **Q20 — Visibility:** the GitHub repo is **public**, with a hard rule that no secrets
  and no real project code ever land in it.

### Amendments discovered while reviewing the plan

- **Node runner label is mandatory, not optional.** The `act_runner` default label maps
  `ubuntu-latest` → `node:16-bullseye`. Node 16 is EOL and **cannot build Angular 22**
  (needs `^20.19.0 || ^22.12.0 || ^24.0.0`, and may have dropped Node 20). Pin the
  runner label to **Node 22 LTS** (`node:22-bookworm`); verify against the sample app's
  `engines` field. Add that image to `images.txt`.
- **`SHARED_NETWORK` is configurable per project.** Compose interpolates the network's
  `name` value, not the map key:
  ```yaml
  networks:
    shared:
      external: true
      name: ${SHARED_NETWORK:-shared-net}
  ```
  The external network must already exist, and the value must match the API team's.
  The stack `.env` sets the machine default; a project's `stack.env` may override.

## Deliverable and scope

**In scope:** the reusable stack (`stack/`), the prep → USB → offline workflow, the
copy-the-files contract, the offline smoke test, the sample Angular 22 app, and the
documentation set.

**Out of scope (now):** a custom ticketing app, offline AI, rootless DinD (documented
as the hardening upgrade path only), multi-machine orchestration.

## Repo layout

```
/
├── stack/                          # reusable, project-independent
│   ├── docker-compose.yml          # gitea, runner, verdaccio, registry, dev
│   ├── .env.example                # SHARED_NETWORK, tokens
│   ├── verdaccio/config.yaml       # uplink present (online) -> removed (offline)
│   ├── prep/
│   │   ├── online.(ps1|sh)         # pull -> save -> warm Verdaccio (idempotent)
│   │   ├── offline.(ps1|sh)        # load -> up (idempotent)
│   │   └── images.txt              # image manifest source of truth
│   └── docs/                       # see "Documentation set"
├── sample-app/                     # disposable Angular 22 fixture
├── artifacts/                      # git-ignored: images.tar, verdaccio storage, MANIFEST
├── docs/adr/                       # ADRs
├── docs/agents/                    # agent skill config (tracker, triage, domain)
├── GLOSSARY.md                     # domain vocabulary
├── brief.md                        # origin
├── AGENTS.md
└── .scratch/                       # this spec and related working docs
```

## The stack (`stack/`)

- Services: **gitea** (1.x, sqlite, Issues + Projects on), **runner** (`act_runner`,
  Node 22 LTS label override, Docker socket), **verdaccio**, **registry:2**, **dev**
  (Node LTS + Angular CLI). All join an external `shared-net` plus an internal `cicd`
  network.
- `shared-net` is the inter-stack contract for talking to the API team's compose; the
  name is interpolated from `SHARED_NETWORK` (see amendment above).
- Version tuple pinned to current stable at build time and recorded, not taken from
  `brief.md`'s suggestions.

## Copy-the-files contract (the "transposable" mechanism)

A new project copies exactly three things into its own repo:

1. **`stack.env`** — app name, image name, port base, `SHARED_NETWORK`, and the
   project's environment list (per project).
2. **`deploy/`** — one small compose template + a per-env launcher.
3. **`.gitea/workflows/`** — the pipeline, parameterized by `stack.env`.

No shared parent repo, no submodule, no scaffolder — a documented three-file contract.

## Environment model

- Generic: any set of environments. **DTAP** ships as the default; a different set is
  just editing `stack.env`.
- Build **once** into an nginx image; **promote the same tag**; environment differences
  (ports, `API_URL`) injected at runtime, never baked in, so one image runs everywhere.

## Prep (online) and offline workflows

- **`prep/online`** (the only internet-touching step): clone this repo (and the target
  project) from GitHub; `docker pull` from `images.txt`; `docker save` →
  `artifacts/images.tar`; start Verdaccio with uplink; `npm ci` through it to warm
  storage; write `MANIFEST`.
- **`prep/offline`**: `docker load`; swap Verdaccio to **no-uplink**; `docker compose
  up`; clone repos into Gitea.
- **Manual offline escape hatches:** hand-drop a `docker load`-able image tar into
  `artifacts/` and hand-place Angular asset files (images/static) into the present app
  — no internet needed.
- **Incremental refresh:** re-running `prep/online` pulls/caches only what's missing;
  the `MANIFEST` diff shows the delta.

## Offline artifact set (USB)

`artifacts/` = `images.tar` + `verdaccio/storage/` + `MANIFEST` (images, tags/digests,
checksums) + git bundles of the stack and the project, plus a version stamp.
Constraint: **no secrets, no real project code** in the public stack repo.

## Verification — the offline smoke test

An automated test that runs the stack with **outbound network blocked** (internal
Docker network), executes the sample app's pipeline end to end, and asserts a deploy
lands in an environment. **This test is the definition of done** for "offline works."
A manual checklist is the fallback. The baseline version tuple is recorded here too.

## Glossary (`GLOSSARY.md`) — candidate terms

Stack · Project · Preparation (Prep) · Artifact set · Manifest · Environment · DTAP ·
Promotion · Copy-the-files contract · Gitea Issues / Board · Runner · Shared network.

## ADRs (`docs/adr/`)

- **0001** Offline prep via `docker save`/`load` + Verdaccio cache (skopeo as upgrade).
- **0002** Generic environment model; one image, promoted; DTAP default.
- **0003** Copy-the-files reuse contract (rejected: shared platform repo / scaffolder).
- **0004** Docker socket in `act_runner` accepted — security caveat, trusted-pusher /
  trusted-artifact rules, trigger to move to rootless DinD.
- **0005** Gitea over GitLab CE (resource trade-off from `brief.md`).
- **0006** Stack runs offline only; online is prep-only.
- **0007** Version policy: pin the stable tuple, don't chase minimums.

## Documentation set (first-class deliverable)

`stack/docs/` + root: `README.md` (overview + quickstart) · `setup-online.md` ·
`offline-install.md` (USB → running) · `onboarding-a-project.md` (the contract) ·
`dtap-and-promotion.md` · `security.md` (**the Docker-socket caveat + rules**) ·
`troubleshooting.md` (Windows/WSL paths, `--poll`, the uplink trap) · `architecture.md` ·
`version-baseline.md`.

## Sequencing (for execution, later)

1. Scaffold layout + `GLOSSARY.md` + the seven ADRs + documentation skeleton.
2. `stack/` compose + config; bring gitea/verdaccio/registry up.
3. Sample Angular 22 app + copy-the-files contract + workflows.
4. Prep/offline scripts + `MANIFEST` + artifacts.
5. Offline smoke test; tune until green.
6. Finish documentation; record the version baseline.

## Open risks

- Docker-socket blast radius (accepted for now; ADR-0004 records the trigger to harden).
- Offline artifact set is part of the trusted computing base (a poisoned dependency or
  image is still code executed with host-Docker power).
- Verdaccio offline only works if the uplink is removed; `maxage` revalidation is a
  known trap (documented in `troubleshooting.md`).
