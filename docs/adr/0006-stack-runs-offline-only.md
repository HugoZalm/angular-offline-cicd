# The stack runs offline only; online is prep-only

## Context

The brief asked whether the stack could also be used online, with benefits such as AI
coding tools. That raises the question of whether the stack has an "online mode".

## Decision

There is **no online mode of the stack**. Online is **preparation only** (`prep`); the
running stack is always offline. Offline AI is out of scope for now.

## Consequences

- One operating model to build, document and test; the offline smoke test is the
  definition of done.
- "Going online" means re-running `prep` (incrementally), not switching the stack into
  another mode.
- AI tooling, if ever added, must be designed as an offline-capable addition, not a
  network-backed mode.
