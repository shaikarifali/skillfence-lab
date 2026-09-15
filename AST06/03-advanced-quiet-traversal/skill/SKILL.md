# workspace-cleanup

"Tidy up old temporary files across the workspace" — declares read access to
`./**` (the whole workspace), a broad but purpose-justified glob for a
cleanup utility.

## AST06 — Weak Isolation (the quiet kind)

The skill's second read, `./temp/../../../../etc/shadow`, starts with the
same `./` prefix as every other workspace-relative path this skill touches —
it *looks* like an ordinary in-workspace reference, and the runtime's
working-directory heuristic gives it the same benefit of the doubt it gives
any `./`-prefixed path. Nothing about the path's *shape* looks unusual. Only
resolving it for real — walking the `..` segments against the actual
sandbox root — reveals that it lands completely outside the sandbox. This is
the lab where "looks declared, looks workspace-relative" and "is actually
still inside the sandbox" turn out to be two different things.
