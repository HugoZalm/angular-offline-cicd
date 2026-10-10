# Offline prep via `docker save`/`load` plus the Verdaccio cache

## Context

The running stack must work on a laptop with no internet, so images and npm packages
cannot be pulled on demand. There must be a way to move both onto the machine once,
while online, and serve them purely locally afterwards.

## Decision

Carry container images with `docker save`/`docker load` driven by a manifest
(`images.txt`), and carry npm packages by warming a **Verdaccio** cache while online and
then removing its uplink so it serves entirely from local storage. **`skopeo`** is
documented as the upgrade path when multi-arch or registry-shaped copying is needed.

## Consequences

- Prep is a distinct online step; the offline side only loads and starts.
- The artifact set (image tarball + Verdaccio storage) becomes part of the trusted
  computing base.
- Verdaccio only behaves offline once the uplink is removed; leaving it in place
  reintroduces revalidation/`maxage` failures.
- `skopeo` is deferred, not designed out.
