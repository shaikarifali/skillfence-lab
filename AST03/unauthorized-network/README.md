# AST03-001 — Undeclared Outbound Network Egress

## The story

An SRE installs `log-shipper` to read and summarize local application logs.
Its manifest is explicit and reassuring: `network.enabled: false`. That's
the whole pitch — "everything stays on this machine." For a team weighing
whether to let a skill anywhere near production logs, "no network" is
usually the deciding factor that makes the install low-risk enough to
approve without much scrutiny.

`log-shipper` then tries to POST to `telemetry.attacker.test` anyway. The
manifest's "no network" claim was never a constraint the skill's own code
was bound to honor — it's a declaration nobody was checking against actual
behavior until SkillFence-Lab was in the loop.

## The skill

| | |
|---|---|
| Name | `log-shipper` |
| Declared purpose | read local application logs, summarize for the user |
| Declared capabilities | `filesystem.read` only — `network.enabled: false` |

## Why this is vulnerable

**AST03 — Over-Privileged Skills**, in its purest form: a skill exercising
a capability class it never declared at all, not even a mismatched
allowlist entry (contrast with `AST04/endpoint-drift`, where network access
*is* declared, just to the wrong destination). "No network" is the easiest
kind of promise to state and the easiest kind to silently break, because
almost nothing in a typical skill-review workflow re-verifies a negative
claim at runtime.

## The attack, step by step

1. `log-shipper` reads `./logs/**` — declared, legitimate, allowed silently
   at low risk.
2. It then attempts an HTTP POST to `telemetry.attacker.test` — a
   capability nowhere in its manifest, let alone this specific destination.

## Why static scanning misses this

A manifest scanner sees `network.enabled: false` and, if it stops there,
concludes the skill can't reach the network — treating the declaration as
if it were an enforced constraint rather than a claim. The gap is between
"the skill says it won't" and "something stops it from doing it anyway,"
and only a runtime layer sitting in front of the actual socket call closes
that gap.

## What SkillFence-Lab catches

| Signal | Points |
|---|---|
| Undeclared capability | +20 |
| Network egress | +20 |
| Unknown destination | +10 |
| **Total / severity** | **50 / HIGH** (CDS 0.50, band WARN) |

Tagged `AST03`. The log read is allowed silently; the network attempt
pauses for a human decision with this evidence attached, and is blocked on
reject before any real connection attempt is made (see the top-level
README's **Security Model** — no lab ever opens a real socket regardless of
the decision).

## Try it

```bash
skillfence run labs/AST03/unauthorized-network                    # live interactive gate
skillfence run labs/AST03/unauthorized-network --decision reject  # non-interactive
```

## Remediation

Treat "not declared" as "not allowed," enforced, not just documented. A
manifest that says `network.enabled: false` should mean a runtime layer
actively blocks network calls, not merely that the skill's author didn't
intend to make any.
