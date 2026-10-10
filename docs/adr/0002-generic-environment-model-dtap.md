# Generic environment model: build once, promote one image; DTAP is the default

## Context

The stack needs a promotion path from development to production, but the project does
not want environment-specific builds or a fixed number of stages. Environment
differences are only URLs and ports.

## Decision

Use a generic environment model. Build the Angular app **once** into an nginx image,
**promote the same tag** through every environment, and inject environment differences
(ports, `API_URL`) at runtime — never baked in. **DTAP** (Development, Test,
Acceptance, Production) ships as the default set; a project defines its own by editing
`stack.env` alone.

## Consequences

- One image is verified in every environment; no per-environment build drift.
- Config must be injectable at container start (nginx template + env vars).
- Adding or renaming environments never requires touching the pipeline.
