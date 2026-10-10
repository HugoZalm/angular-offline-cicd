# Security

The Docker-socket caveat and the rules that follow from it
([ADR-0004](../adr/0004-docker-socket-in-act-runner.md)).

> Status: skeleton. Full rules are written by the documentation ticket.

## The caveat

The Docker socket is mounted into `act_runner` so jobs can build and deploy containers.
That gives jobs **root-equivalent control of the host** — and the offline _artifact set_
is part of the trusted computing base.

## Rules

_TODO:_

- **Trusted pusher** — only trusted people may push to the repos.
- **Trusted artifact** — only trusted images/dependencies enter the pipeline.
- No secrets in the public stack repo.

## When to harden

Move to rootless Docker-in-Docker when _TODO: trigger condition_.
