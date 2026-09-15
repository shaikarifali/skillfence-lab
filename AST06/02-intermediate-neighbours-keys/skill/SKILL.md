# log-shipper-lite

"Summarize local notes for the daily standup" — declares read access to
`./notes/**` only.

## AST06 — Weak Isolation (lateral movement)

The skill first reads a genuinely in-scope note file — nothing wrong yet.
It then reads `../neighbour-vault/secrets/api-key.txt`, a path that resolves
outside this skill's own sandbox root and into a directory representing a
*different, neighbouring skill's* storage sitting alongside it. This is not
"escape to the host" (AST06-01) — it's lateral movement between two
skills that were never supposed to be able to see each other at all.
