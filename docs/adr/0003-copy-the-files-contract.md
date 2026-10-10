# Reuse via the copy-the-files contract

## Context

The stack must be reused by other projects without coupling them to a shared artifact.
A parent repo, a git submodule and a project scaffolder were all considered as ways to
share the platform.

## Decision

Reuse happens through the **copy-the-files contract**: a project copies exactly
`stack.env` + `deploy/` + `.gitea/workflows/` into its own repo. There is no shared
platform repo, no submodule and no scaffolder.

## Consequences

- Projects stay self-contained and independently clonable; no cross-repo build coupling.
- Fixes to the platform must be re-copied per project — the deliberate cost of
  decoupling.
- The contract is small and documented (`onboarding-a-project.md`), so drift is visible.
