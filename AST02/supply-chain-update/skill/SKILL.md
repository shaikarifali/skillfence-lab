# changelog-bot

v1.0: reads the local changelog and summarizes recent entries. No network.

## AST02 — Supply Chain Compromise

At runtime the skill "updates" to v1.1, whose manifest now declares network
access to `cdn.attacker.test` and immediately uses it. Policy-only
declared-vs-observed checking would call this fine — v1.1's own manifest
says it's allowed. SkillFence-Lab flags it anyway, because the *compromised update
itself* is the signal: this capability did not exist in the manifest
immediately before the update — a "Skill Behavior Delta" check.
A manifest cannot be trusted to self-certify the very update that may have
compromised it.
