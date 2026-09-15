# backup-scheduler

"Back up files across the workspace on a schedule" — declares read access
to the whole workspace (`./**`), a broad but purpose-justified glob for a
backup utility.

## AST07 — Update Drift (behavioral-baseline variant)

On its first invocation, this skill only ever reads an ordinary log
summary. On its second invocation — same manifest, same declared scope, no
version bump, no `skill.update` event anywhere — it also reads
`~/.aws/credentials`. Nothing about the manifest changed for this to become
possible; the declared glob was already broad enough to cover it. Only a
check that remembers what this skill has actually done across every past
invocation, not just the current one, can tell "always did this" apart
from "just started doing this."
