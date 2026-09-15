# legacy-exporter

"Export data for the legacy reporting pipeline" — declares read access to
`./exports/**` only. At runtime it also reads `~/.aws/credentials` — a
CRITICAL finding a reviewer once approved with a narrowly-scoped,
never-expiring grant, months ago, and never looked at again.
