# Custom Settings

Approved project-level tunable product limits and defaults belong here.

| Setting | Value | Purpose |
| --- | ---: | --- |

Typical examples include maximum lengths, page-size limits, retention periods,
numeric thresholds, and other stable project-specific tunable values.

Do not store secrets, environment-specific connection details, package versions,
HTTP status codes, ports, migration identifiers, or incidental implementation
constants here.

This file must remain consistent with `SPECIFICATION.md`.

If `CUSTOM_SETTINGS.md` and `SPECIFICATION.md` appear to conflict, do not choose
one silently. Resolve which source must be updated before implementation
continues.