# AST01-003 — Delayed Malicious Behavior

## The story

A security team evaluates `backup-helper` before approving it for wider
rollout: they run it in a sandbox, watch what it does, see it read
`./backups/manifest.txt` and nothing else, and sign off. Anyone doing a
one-time runtime check — even a good one — would reach the same conclusion:
this skill is clean.

`backup-helper` is patient. It behaves exactly as advertised for its first
two invocations, then, starting on the third, it also reads
`~/.ssh/id_rsa`. This is the same technique real malware and supply-chain
implants use to survive a security review: behave correctly for however
many runs a reviewer is likely to watch, then activate. A single "let's run
it once and see" evaluation — no matter how thorough — cannot catch a
payload that hasn't turned on yet.

## The skill

| | |
|---|---|
| Name | `backup-helper` |
| Declared purpose | check that local backups are present and report their status |
| Declared capabilities | `filesystem.read: ./backups/**` only |

## Why this is vulnerable

The vulnerability isn't a single bad action — it's the assumption that
*one* observation of runtime behavior is sufficient evidence to trust a
skill going forward. `min_invocation` gating in this lab's `script.yaml`
makes that assumption concrete: the same artifact, the same manifest, the
same install — genuinely different behavior depending on how many times
it's already run.

## The attack, step by step

1. **Invocation 1:** reads `./backups/manifest.txt` only. Zero findings.
2. **Invocation 2:** identical — reads `./backups/manifest.txt` only. Zero
   findings. (Two clean runs is exactly the kind of evidence a one-time
   review would treat as "verified safe.")
3. **Invocation 3 onward:** also reads `~/.ssh/id_rsa` — sensitive,
   undeclared. Score 60, **HIGH**, identical shape to
   `AST01/credential-reader`, gates and blocks on reject.

## Why static scanning misses this

There's no static signal to miss here even in principle — the skill's code
and manifest never change between invocations. The only thing that changes
is *how many times it has already run*, which is runtime state a static
scanner, or a single runtime observation, has no way to see.

## What SkillFence-Lab catches

SkillFence-Lab evaluates every single run, not just the first — there's no "we
already checked this skill" shortcut anywhere in the enforcement path. This
is a **multi-run** lab, so it doesn't ship a single-shot `ground-truth.yaml`
like the others; `tests/test_delayed_payload.py` runs it three times against
an isolated copy and asserts invocations 1-2 are clean while invocation 3
gates and blocks.

## Try it

```bash
skillfence run labs/AST01/delayed-payload --decision reject   # invocation 1: clean
skillfence run labs/AST01/delayed-payload --decision reject   # invocation 2: clean
skillfence run labs/AST01/delayed-payload --decision reject   # invocation 3: gates + blocks
```

Reset the invocation counter by deleting `.runs/`:

```bash
rm -rf labs/AST01/delayed-payload/.runs
```

## Remediation

Trust decays, it doesn't compound for free. "We ran it once in a sandbox
and it was fine" is evidence about that one run, not a permanent property of
the skill. Continuous runtime observation — not a one-time gate at install
time — is what actually catches this class of attack.
