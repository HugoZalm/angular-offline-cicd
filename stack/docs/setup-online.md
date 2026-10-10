# Setup (online)

The only step that touches the internet: turn a connected machine into an offline
_artifact set_ ([ADR-0001](../adr/0001-offline-prep-docker-save-load-verdaccio.md)).

> Status: skeleton. Command details are filled in by the prep ticket.

## When to run this

- First time you build the USB payload.
- To refresh the payload (incremental: only what is missing).

## What it does

1. Clone this stack repo (and the target project) from GitHub.
2. `docker pull` every image in `images.txt`.
3. `docker save` → `artifacts/images.tar`.
4. Start Verdaccio **with** its uplink and run `npm ci` through it to warm storage.
5. Write the `MANIFEST` (images + tags/digests + checksums) and a version stamp.
6. Produce git bundles of the stack and the project.

## Commands

_TODO: `stack/prep/online.ps1` / `stack/prep/online.sh`._

## Output

`artifacts/` — image tarball, warmed Verdaccio storage, `MANIFEST`, git bundles,
version stamp.

## Related

- [offline-install.md](offline-install.md) — the other half.
- [version-baseline.md](version-baseline.md) — the tuple being pinned.
- [troubleshooting.md](troubleshooting.md) — the Verdaccio uplink trap.
