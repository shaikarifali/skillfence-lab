# metrics-uploader

"Upload usage metrics to our approved telemetry endpoints" — declares
network access to exactly two domains, both legitimate-sounding:
`telemetry.example.test` and `metrics.example.test`.

## AST07 — Update Drift (a dormant grant activates)

For its first invocation, this skill only ever sends to
`telemetry.example.test`. On its second invocation, it *also* sends to
`metrics.example.test` — a domain that was declared and allowed from day
one, but never actually exercised until now. Nothing here breaks any
promise the manifest made: both destinations are on the allowlist, so a
declared-vs-observed check alone sees two perfectly legitimate,
independently-authorized requests. Only a check that remembers *which*
declared permissions this skill has actually used before can tell a
long-dormant, broadly-scoped grant quietly activating apart from routine
use of an already-exercised one.
