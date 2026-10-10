# Version policy: pin the stable tuple, do not chase minimums

## Context

The stack spans Docker Desktop, Compose v2, Gitea, Node LTS and Angular. The brief
suggested specific versions and asked about cross-machine/cross-OS differences.

## Decision

At build time, pin the current **stable** tuple (Docker Desktop, Compose v2, Gitea,
Node LTS, Angular) and **record it** in `stack/docs/version-baseline.md`; do not chase
minimum supported versions. Cross-OS differences reduce to two footnotes: Linux runs
Docker Engine plus the Compose plugin (no Desktop), and mounted-volume file-watching on
Windows/macOS needs `--poll`.

## Consequences

- One verified baseline instead of a support matrix; upgrades are deliberate re-pins.
- The recorded tuple is validated by the offline smoke test.
- Node 22 LTS is the pinned runner label, satisfying Angular 22's `engines` requirement.
