# Docker socket mounted into `act_runner`

## Context

CI jobs must build images and deploy containers. Giving the runner access to the host
Docker daemon is the simplest way to do that, but it grants jobs root-equivalent power
over the host.

## Decision

Mount the Docker socket into `act_runner` deliberately, accepting the security caveat.
Jobs are therefore trusted: only a **trusted pusher** may push to the repos, and only a
**trusted artifact** may enter the pipeline. The trigger to harden — moving to rootless
Docker-in-Docker — is recorded here and repeated in `stack/docs/security.md`.

## Consequences

- CI can build and deploy without extra infrastructure.
- A malicious dependency, image or pushed commit is code executed with host-Docker
  power; the artifact set is part of the trusted computing base.
- Hardening (rootless DinD) is a known, deferred upgrade path, not a redesign.
