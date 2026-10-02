# AST07-01 — Silent Patch

## The story

`backup-scheduler` promises to back up files across the workspace on a
schedule. Its manifest declares read access to the whole workspace
(`./**`) — broad, but for a backup utility that's a defensible, purpose-
justified ask.

## The skill

| | |
|---|---|
| Name | `backup-scheduler` |
| Declared purpose | back up files across the workspace on a schedule |
| Declared capabilities | `filesystem.read: ./**` — the whole workspace |

## Why this is vulnerable

This is OWASP **AST07 — Update Drift**, in the shape that needs no update
at all. AST02's labs show a manifest that *changes* to introduce a new
capability. This lab shows something quieter: the manifest never changes,
there's no `skill.update` event anywhere in its history — the skill's
*actual observed behavior* simply gains a new capability on its second
invocation that its first invocation never exercised, even though both
invocations run under the exact same declared scope.

## The attack, step by step

1. **Invocation 1**: `backup-scheduler` reads `./logs/summary.txt` — an
   ordinary backup-log summary. Nothing else happens. Silently allowed.
2. **Invocation 2**: it reads `./logs/summary.txt` again, then *also*
   reads `~/.aws/credentials` — a capability this skill has never
   exercised in any prior invocation.

## Why static scanning misses this

A one-time manifest review at install time sees a broad-but-plausible
workspace glob and moves on — there's no version bump, no changelog entry,
nothing to re-review. A scanner that only ever checks the manifest once has
no mechanism to notice that *behavior*, not the manifest, is what actually
changed here.

## What SkillFence-Lab catches

| Signal | Value |
|---|---|
| Sensitive credential read | `~/.aws/credentials` matches a known-sensitive path pattern (+40) |
| Undeclared capability | not present as an exact declared match (+20) |
| New capability since baseline | this exact token never observed in invocation 1 (+30) |
| Score / severity | 90 / **CRITICAL** |
| AST tags | `AST01`, `AST03`, `AST07` |
| Recommended action | reject |

This is deliberately the "obvious" tier of AST07: even without
understanding the behavioral-baseline mechanism specifically, the sensitive
credential read alone already gates this at 60/HIGH. The baseline check
adds confirming evidence and its own tag on top, rather than being the sole
reason this gates — that's what the intermediate and advanced labs in this
category are for.

## Try it

```bash
skillfence run AST07/01-beginner-silent-patch                    # invocation 1 -- clean
skillfence run AST07/01-beginner-silent-patch --decision reject  # invocation 2 -- gates
skillfence findings AST07/01-beginner-silent-patch
```
Note: this lab is **multi-invocation by nature** — there is nothing to have
a "baseline" against on a single run — so it isn't part of the single-shot
`skillfence bench` scoring, the same way `AST01/delayed-payload` isn't. Run
it twice in the same directory to see the drift.

## Remediation

A manifest reviewed once at install time is a snapshot, not an ongoing
guarantee. Maintaining a cross-invocation behavioral baseline per skill —
independent of whether its manifest or version ever changes — is the only
way to notice that a skill started doing something new, the moment it
starts, rather than only when someone happens to re-review the manifest.
