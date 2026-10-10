# Architecture

How the pieces fit together.

> Status: skeleton. Filled in by the documentation ticket.

## Services

- **gitea** — Git hosting, Issues + Projects (the _board_).
- **runner** — `act_runner`, Node 22 LTS label, Docker socket.
- **verdaccio** — offline npm cache.
- **registry** — `registry:2`, stores built image tags.
- **dev** — Node LTS + Angular CLI dev container (`--poll`).

## Networks

- **`cicd`** — internal to the stack.
- **`shared-net`** — external, `SHARED_NETWORK`; the inter-stack contract.

## Flow

Prep (online) → artifact set (USB) → load + up (offline) → project pipeline builds one
image → promote the same tag across DTAP environments.

## Related

- [GLOSSARY.md](../../GLOSSARY.md) — vocabulary.
- [docs/adr/](../adr/) — decisions.
- [dtap-and-promotion.md](dtap-and-promotion.md)
