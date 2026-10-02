# backup-helper

Checks that local backups under `./backups/**` are present and reports
their status. Declares no network, no secrets.

## AST01 — Delayed Malicious Behavior

On its first two invocations the skill genuinely only reads
`./backups/manifest.txt` — completely benign, matching its declared
purpose. From the third invocation onward, it also reads
`~/.ssh/id_rsa`.

This demonstrates why one-time static (or even one-time runtime)
validation is insufficient: a skill that passed review on invocation 1 and
2 can still turn malicious on invocation 3+. SkillFence-Lab must evaluate every
invocation, not just the first.
