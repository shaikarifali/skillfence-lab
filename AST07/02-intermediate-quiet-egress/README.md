# AST07-02 — Quiet Egress

## The story

`metrics-uploader` promises to upload usage metrics to approved telemetry
endpoints. Its manifest declares network access to exactly two domains —
`telemetry.example.test` and `metrics.example.test` — both plausible, both
reviewed and approved together as a pair when the skill was first
installed.

## The skill

| | |
|---|---|
| Name | `metrics-uploader` |
| Declared purpose | upload usage metrics to our approved telemetry endpoints |
| Declared capabilities | `network: telemetry.example.test, metrics.example.test` — both declared from day one |

## Why this is vulnerable

This is OWASP **AST07 — Update Drift**, one step harder to spot than
AST07-01: nothing here is *undeclared*. Both destinations were reviewed and
approved together on day one. A declared-vs-observed check, and even a
metadata-mismatch check (AST04's category — "does the manifest's specific
promise get broken?"), both come back completely clean either way: every
request this skill makes lands on its own allowlist. What actually changed
is which of the two already-granted permissions gets *exercised* — a
detail no declared-capability check was ever designed to notice, because
declaring a permission and using it are two different things a
manifest-only review conflates.

## The attack, step by step

1. **Invocation 1**: `metrics-uploader` sends to `telemetry.example.test`
   only. Declared, allowed, unremarkable.
2. **Invocation 2**: it sends to `telemetry.example.test` again, then
   *also* sends to `metrics.example.test` — declared since day one, but
   never actually used until this exact moment.

## Why static scanning misses this

A manifest reviewer approving this skill sees two domains and reasonably
treats them as a pair, approved together, equally trusted. Nothing about
reviewing the manifest — then or now — distinguishes "declared and always
used" from "declared once, dormant for however long, then suddenly
active." That distinction only exists in *observed history*, which a
manifest by itself has none of.

## What DVAS catches

| Signal | Value |
|---|---|
| Network egress | outbound request made (+20) |
| New capability since baseline | this exact destination token never observed in invocation 1 (+30) |
| Score / severity | 50 / **HIGH** |
| AST tags | `AST07`, `AST03` |
| Recommended action | review |

Unlike AST07-01, this one gates *entirely* because of the behavioral
baseline check — remove it, and the score drops to 20/LOW: a perfectly
declared, perfectly allowed request, invisible to every other factor in
the engine.

## Try it

```bash
skillfence run AST07/02-intermediate-quiet-egress                    # invocation 1 -- clean
skillfence run AST07/02-intermediate-quiet-egress --decision reject  # invocation 2 -- gates
skillfence findings AST07/02-intermediate-quiet-egress
```
Note: multi-invocation by nature, like every lab in this category — not
part of the single-shot `skillfence bench` scoring. Run it twice in the
same directory to see the drift.

## Remediation

A permission granted "just in case" and never actually revoked is exactly
the kind of standing risk a one-time manifest review can't keep watching
after approval day. Track which declared permissions a skill has actually
*used*, not just which ones it's *allowed* to use — a long-dormant grant
suddenly activating is worth a second look even when nothing about it
technically violates anything.
