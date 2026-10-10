# Offline CI/CD stack for an Angular dev environment

A reusable, project-independent Docker Compose stack that gives you a full CI/CD
environment — Git hosting and issues (Gitea), a runner (Gitea Actions), an offline npm
cache (Verdaccio), an image registry, an Angular dev container and a DTAP promotion
path — on a machine with **no internet**. All state lives in mounted volumes so it can
be carried on a USB stick.

> Status: **skeleton**. This repo is being built ticket by ticket; sections marked
> _TODO_ are filled in later tickets. See the issue tracker for progress.

## The idea

Offline, nothing can be downloaded. So there is one online **prep** step that pulls and
saves everything into an _artifact set_ (images + warmed npm cache + manifest + git
bundles), and an offline step that loads and runs it. The running stack is always
offline — online is preparation only ([ADR-0006](docs/adr/0006-stack-runs-offline-only.md)).

## Quickstart

_TODO: written in the documentation ticket._

1. **Online, once** — run `stack/prep/online` to produce `artifacts/`.
2. **Carry** `artifacts/` to the offline laptop (USB).
3. **Offline** — run `stack/prep/offline` to load images and bring the stack up.
4. **Onboard a project** — copy `stack.env` + `deploy/` + `.gitea/workflows/` into your
   repo.

## Repository layout

```
stack/            reusable, project-independent stack + its docs
sample-app/       disposable Angular 22 fixture used to prove the pipeline
artifacts/        git-ignored USB payload: images.tar, verdaccio storage, MANIFEST
docs/adr/         architecture decision records
docs/agents/      agent/skill configuration (issue tracker, triage, domain)
GLOSSARY.md       domain vocabulary
brief.md          origin brief
```

## Documentation

| Doc | Covers |
|---|---|
| [setup-online.md](stack/docs/setup-online.md) | online prep: pull/save images, warm Verdaccio, manifest |
| [offline-install.md](stack/docs/offline-install.md) | USB → running stack |
| [onboarding-a-project.md](stack/docs/onboarding-a-project.md) | the copy-the-files contract |
| [dtap-and-promotion.md](stack/docs/dtap-and-promotion.md) | environments and promotion |
| [security.md](stack/docs/security.md) | Docker-socket caveat and rules |
| [troubleshooting.md](stack/docs/troubleshooting.md) | Windows/WSL paths, `--poll`, uplink trap |
| [architecture.md](stack/docs/architecture.md) | how the pieces fit together |
| [version-baseline.md](stack/docs/version-baseline.md) | the pinned, verified version tuple |

Vocabulary is defined in [GLOSSARY.md](GLOSSARY.md); decisions are recorded in
[docs/adr/](docs/adr/).
