# Gitea over GitLab CE

## Context

The stack needs Git hosting plus issues, labels, milestones, a kanban board and CI. The
main open-source candidates are Gitea and GitLab CE; the target machine is a laptop.

## Decision

Choose **Gitea** with Gitea Actions. GitLab CE's all-in-one feature set was rejected
because it needs roughly 4 GB+ RAM and is too heavy for a laptop.

## Consequences

- Low resource use (on the order of ~200 MB RAM) suits the laptop target.
- CI is GitHub-Actions-style YAML via `act_runner`, not GitLab CI.
- Any GitLab-only feature is out of reach; the trade-off is accepted.
