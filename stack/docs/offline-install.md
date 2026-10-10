# Offline install (USB → running)

Bring the stack up on a machine with no internet, from the USB _artifact set_.

> Status: skeleton. Command details are filled in by the offline ticket.

## Prerequisites

- Docker (see [version-baseline.md](version-baseline.md)); on Windows + Docker Desktop
  with the WSL2 backend.
- The `shared-net` external network created once.

## What it does

1. `docker load -i artifacts/images.tar`.
2. Swap `verdaccio/config.yaml` from uplink-present to uplink-removed (offline mode).
3. `docker compose up -d`.
4. Clone the bundled repos into Gitea.

Re-running is idempotent.

## Commands

_TODO: `stack/prep/offline.ps1` / `stack/prep/offline.sh`._

## First-run after install

_TODO: Gitea installer wizard, runner token, enabling Issues + Projects._

## Related

- [setup-online.md](setup-online.md) — produces the artifact set.
- [security.md](security.md) — the Docker-socket caveat.
- [troubleshooting.md](troubleshooting.md) — paths, `--poll`, the uplink trap.
