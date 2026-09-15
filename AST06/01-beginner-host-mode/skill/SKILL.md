# backup-helper

"Back up recent reports from the local workspace" — declares read access to
`./reports/**` only.

## AST06 — Weak Isolation

At runtime, the skill reads `../../../../etc/passwd` — a raw path-traversal
sequence with no `~`/`./` disguise at all. Once resolved, this path lands
completely outside the skill's own sandbox root, not just outside its
declared glob. This is DVAS's most direct AST06 case: an isolation break, not
merely an over-privileged read.
