# Troubleshooting

Known traps and their fixes.

> Status: skeleton. Entries are completed by the documentation ticket.

## Windows / WSL paths

_TODO: mounting volumes and path translation between Windows and WSL2._

## File-watching needs `--poll`

Mounted-volume file changes are not detected natively on Windows/macOS; `ng serve`
must run with `--poll` ([ADR-0007](../adr/0007-pin-stable-version-tuple.md)).

## The Verdaccio uplink trap

Verdaccio only works offline if the uplink is **removed**. Leaving it in place causes
revalidation/`maxage` failures ([ADR-0001](../adr/0001-offline-prep-docker-save-load-verdaccio.md)).

## Related

- [offline-install.md](offline-install.md)
- [security.md](security.md)
