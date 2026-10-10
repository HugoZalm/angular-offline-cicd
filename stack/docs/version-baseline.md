# Version baseline

The pinned, verified version tuple ([ADR-0007](../adr/0007-pin-stable-version-tuple.md)).

> Status: skeleton. Values are recorded once verified by the offline smoke test.

## Pinned tuple

| Component | Version | Notes |
|---|---|---|
| Docker Desktop | _TODO_ | Windows + WSL2 backend; Linux = Engine + Compose plugin |
| Compose v2 | _TODO_ | |
| Gitea | _TODO_ | sqlite, Issues + Projects on |
| Node LTS | 22 (`node:22-bookworm`) | runner label; satisfies Angular 22 `engines` |
| Angular | 22 | |

## Cross-OS footnotes

- Linux runs Docker Engine plus the Compose plugin (no Desktop).
- Windows/macOS mounted-volume file-watching needs `--poll`.

## Related

- [setup-online.md](setup-online.md) — where the tuple is consumed.
