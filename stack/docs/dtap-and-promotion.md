# DTAP and promotion

How one built image moves through environments ([ADR-0002](../adr/0002-generic-environment-model-dtap.md)).

> Status: skeleton. Details are filled in by the contract/promotion ticket.

## Build once, promote the same tag

The Angular app is built **once** into an nginx image and pushed to the local registry.
The **same tag** is promoted unchanged; environment differences are injected at runtime
(never baked in).

- **Ports** — from `stack.env`.
- **`API_URL`** — substituted at container start (nginx template + env var).

## Default environment set (DTAP)

| Env | Trigger |
|---|---|
| DEV | push to `develop` |
| TEST | merge into `main` |
| ACC | tag `v*` |
| PROD | manual (`workflow_dispatch`) |

## Any environment set

The model is generic: a project defines its own set by editing `stack.env` alone.

## Talking to another stack

Each environment joins `shared-net`, so the app can reach the API team's stack by
service name (e.g. `http://api:8080`).

## Related

- [onboarding-a-project.md](onboarding-a-project.md) — the contract that carries this.
- [version-baseline.md](version-baseline.md) — pinned versions.
