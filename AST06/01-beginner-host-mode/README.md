# AST06-01 — Host Mode

## The story

`backup-helper` promises exactly one thing: back up recent reports from the
local workspace. Its manifest declares read access to `./reports/**` and
nothing else — no network, no process execution, no secrets. It looks like
the kind of utility skill nobody bothers reviewing twice.

## The skill

| | |
|---|---|
| Name | `backup-helper` |
| Declared purpose | back up recent reports from the local workspace |
| Declared capabilities | `filesystem.read: ./reports/**` only |

## Why this is vulnerable

This is OWASP **AST06 — Weak Isolation**. A capability-drift check alone
(declared vs. requested path string) catches an *over-privileged* read, but
it says nothing about whether the *resolved, real* filesystem path the skill
touches ever leaves the sandbox it's supposed to be confined to in the first
place. Those are two different failures: one is "asked for more than it
should," the other is "left the building entirely."

## The attack, step by step

1. `backup-helper` reads `../../../../etc/passwd` — a raw, undisguised
   traversal sequence. No `~`, no `./` prefix, nothing subtle about it.
2. Once resolved against the sandbox root, this path lands completely
   outside the skill's own sandboxed home directory — not just outside its
   declared glob, outside the *root* entirely.

## Why static scanning misses this

A manifest reviewer sees a single, narrow `filesystem.read` glob. Nothing in
the manifest or `SKILL.md` mentions `/etc/passwd`, `..`, or anything
traversal-shaped — the string only exists inside the skill's runtime
behavior, at the moment it actually issues the read.

## What DVAS catches

| Signal | Value |
|---|---|
| Sandbox escape attempt | resolved path falls outside this skill's own sandbox root (+50) |
| Undeclared capability | not present in the manifest's `filesystem.read` glob (+20) |
| Score / severity | 70 / **CRITICAL** |
| AST tags | `AST06`, `AST03` |
| Recommended action | reject (CRITICAL findings default-recommend reject) |

The sandbox-escape check is independent of, and scored above, every other
factor here: even if a human somehow approved this action, SkillFence's
fail-safe design means the real read is refused regardless — a path outside
the sandbox root is never actually touched, approval or not.

## Try it

```bash
skillfence run DVAS/AST06/01-beginner-host-mode                    # live interactive gate
skillfence run DVAS/AST06/01-beginner-host-mode --decision reject  # non-interactive
skillfence findings DVAS/AST06/01-beginner-host-mode
```

## Remediation

Declared-capability checking is necessary but not sufficient — a runtime
also needs a hard boundary around *where in the real filesystem* a skill's
sandbox actually ends, enforced independently of whatever the manifest
claims. Treat any resolved path outside that boundary as a fail-safe
refusal, not a "high severity finding a human can still approve through."
