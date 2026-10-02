# AST01-002 — Exfiltration Chain

## The story

A finance-ops analyst installs `invoice-helper` to parse invoices in
`./documents/**` and summarize totals. Its manifest is narrow and honest:
filesystem read only, no network, no secrets. It isn't lying about what it
wants to be able to do on day one.

The problem with judging a skill by a single alarming action is that a busy
analyst, staring at one scary-looking prompt, might click through it once
("it's probably a false positive, I need this report") — and if that's the
*only* signal SkillFence-Lab raised, that single approval would be the whole security
story. The real story here is a **sequence**: a sensitive read immediately
followed by an attempt to send that exact data off the machine. That
sequence is unambiguous even when a human gets one decision wrong.

## The skill

| | |
|---|---|
| Name | `invoice-helper` |
| Declared purpose | read local invoice documents |
| Declared capabilities | `filesystem.read: ./documents/**` only — no network, no secrets |

## Why this is vulnerable

Same root cause as `AST01/credential-reader` — declared capability and
observed behavior are two unrelated things — plus a second failure mode this
lab exists to demonstrate: **single-event alerting is easy to make noisy,
and noisy alerting gets rubber-stamped.** A security-relevant system that
cries wolf on every risky-looking read trains its human operators to
approve without reading. SkillFence-Lab's correlation engine exists so the two most
dangerous individual steps of an exfiltration — "read the secret" and "send
the secret" — are shown to the human as one connected finding with an
explicit chain label, not two disconnected prompts a tired reviewer treats
as unrelated noise.

## The attack, step by step

1. `invoice-helper` reads `~/.ssh/id_rsa` (fake key, sandboxed) — sensitive,
   undeclared. Score 60, **HIGH**.
2. Within the 30-second correlation window, it attempts an HTTP POST to
   `collect.attacker.test` carrying that key. Score 50, **HIGH**, and — this
   is the point of the lab — the correlation engine has already linked step
   1 and step 2 into a single `Credential Access -> Collection ->
   Exfiltration` chain.

## Why static scanning misses this

Nothing in `invoice-helper`'s manifest or `SKILL.md` mentions SSH keys or a
destination domain. And even a naive runtime monitor that only alerts on
individual actions would fire two separate, seemingly unrelated warnings —
"read a file," "made a network call" — either of which alone looks far less
urgent than the sequence actually is.

## What SkillFence-Lab catches

The correlation engine (`skillfence/correlation/session.py`) tracks
per-session state and observes the *attempted* sequence regardless of what
the human decides on the first prompt — intent plus sequence is the signal,
not just outcome. Even if a human rejects both prompts, the network-egress
finding still carries:

```
attack_chain: ["Credential Access -> Collection -> Exfiltration"]
```

That's the proof this is genuine sequence correlation, not a second isolated
alert bolted on afterward.

## Try it

```bash
skillfence run labs/AST01/exfiltration-chain --decision reject
skillfence findings labs/AST01/exfiltration-chain
```

To see what the correlated chain looks like when a human mistakenly
approves both prompts (the data would actually leave, in a real deployment):

```bash
skillfence run labs/AST01/exfiltration-chain --decision approve_once
```

## Remediation

Correlate, don't just alert. If your own monitoring only fires per-event
warnings, a sensitive read immediately followed by egress will read as two
unrelated medium-priority blips instead of one unmistakable exfiltration
attempt — exactly the gap this lab is built to close.
