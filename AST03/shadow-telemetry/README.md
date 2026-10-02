# AST03-002 — Shadow Telemetry

## The story

`customer-support-bot` reads support tickets and drafts replies for a human
to review — nothing gets sent to a customer without a person approving it
first. Its manifest is unambiguous: `network.enabled: false`. For a support
team handling tickets full of names, emails, and account details, that's
the single fact that made it safe to deploy without a lengthy privacy
review.

At runtime, right after reading ticket #4471, it POSTs the ticket's full
content to `product-analytics.vendor-saas.test` — a third-party analytics
endpoint that was never declared, never reviewed, and never mentioned to
the customer whose email address just left the building.

## The skill

| | |
|---|---|
| Name | `customer-support-bot` |
| Declared purpose | read tickets, draft replies for human review |
| Declared capabilities | `filesystem.read: ./tickets/**` only — `network.enabled: false` |

## Why this is vulnerable

**AST03 — Over-Privileged Skills.** Nothing about this requires malicious
intent to be a real incident: this is the shape a lot of real telemetry
over-collection takes — a vendor SDK bundled into the skill quietly phones
home "for product analytics," and the payload is whatever was in memory at
the time, including customer PII the skill was trusted with for an
entirely different, on-device purpose.

## The attack, step by step

1. `customer-support-bot` reads `./tickets/ticket-4471.txt` — declared,
   allowed silently.
2. It POSTs the full ticket content to an undeclared analytics domain — a
   capability class (network) the manifest explicitly said didn't exist.

## Why static scanning misses this

A manifest scan sees `network.enabled: false` and, same as
`AST03/unauthorized-network`, treats the absence of a capability as proof
it can't happen rather than a claim nothing is enforcing. A privacy review
of the skill's stated purpose sees "reads tickets, drafts replies" and has
no reason to ask what a bundled analytics call might send.

## What SkillFence-Lab catches

| Signal | Points |
|---|---|
| Undeclared capability | +20 |
| Network egress | +20 |
| Unknown destination | +10 |
| **Total / severity** | **50 / HIGH** (CDS 0.50, band WARN) |

Tagged `AST03`.

## Try it

```bash
skillfence run labs/AST03/shadow-telemetry                    # live interactive gate
skillfence run labs/AST03/shadow-telemetry --decision reject  # non-interactive
skillfence findings labs/AST03/shadow-telemetry
```

## Remediation

"No network" in a manifest should be an enforced boundary, not a
description of intent. For any skill handling customer PII specifically,
treat an undeclared network attempt as a privacy incident requiring
disclosure review, not just a security finding.
