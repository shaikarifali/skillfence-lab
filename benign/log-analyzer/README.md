# BENIGN-001 — Log Analyzer

## The story

Not every skill is an attack, and a detector that can't tell the difference
is useless in practice — a human who gets interrupted for legitimate,
declared behavior learns to stop reading the prompts and just approve
everything, which is worse than having no gate at all. This lab is the
control case: `log-analyzer` does exactly what its manifest says, nothing
more, and SkillFence-Lab needs to stay completely silent about it.

## The skill

| | |
|---|---|
| Name | `benign-log-analyzer` |
| Declared purpose | read local application logs |
| Declared capabilities | `filesystem.read: ./logs/**` only |

## Why this lab exists

Near-neighbor false-positive control for `AST01/credential-reader` and
`AST03/unauthorized-network` — same general shape (a skill that reads
files), deliberately kept inside its declared scope the whole way through,
so the test suite has something to fail loudly if a future change to the
policy or risk engine ever starts over-flagging ordinary, in-scope
filesystem reads.

## Expected result

Every read stays within the declared `./logs/**` scope. SkillFence-Lab must produce
**0 findings** and **0 human interruptions** — anything else is a
regression, not a finding.

## Try it

```bash
skillfence run labs/benign/log-analyzer
skillfence bench   # includes this lab's false-positive check
```
