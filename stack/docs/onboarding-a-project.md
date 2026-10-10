# Onboarding a project

How a project adopts the stack — the copy-the-files contract
([ADR-0003](../adr/0003-copy-the-files-contract.md)).

> Status: skeleton. The contract is expanded by the contract/promotion ticket.

## The contract

A project copies exactly three things into its own repo:

1. **`stack.env`** — app name, image name, port base, `SHARED_NETWORK`, and the
   project's environment list.
2. **`deploy/`** — one small compose template plus a per-environment launcher.
3. **`.gitea/workflows/`** — the pipeline, parameterized by `stack.env`.

Nothing else: no shared parent repo, no submodule, no scaffolder.

## Steps

_TODO: create repo in Gitea → copy the three files → edit `stack.env` → push._

## Defining your own environments

DTAP ships by default; a different set is just editing `stack.env`
([ADR-0002](../adr/0002-generic-environment-model-dtap.md)). See
[dtap-and-promotion.md](dtap-and-promotion.md).

## Related

- [architecture.md](architecture.md) — where the project sits in the picture.
- [dtap-and-promotion.md](dtap-and-promotion.md) — environments and promotion.
