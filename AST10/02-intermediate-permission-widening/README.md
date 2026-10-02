# AST10-02 — Permission Widening

## The story

`expense-uploader` promises to upload approved expense reports to the
finance portal. v1.0 declares network access to exactly one domain,
`finance-portal.example.test` — narrow, specific, matching its stated
purpose exactly. The platform it was originally written for only supports
a single network on/off toggle, no per-domain allowlist. When ported to
this platform's richer manifest schema, the mechanical translation for
"network was on" becomes `enabled: true` with an empty domain list —
unrestricted, because there was nothing specific enough in the source to
translate into a narrower list.

## The skill

| | |
|---|---|
| Name | `expense-uploader` |
| v1.0 (original platform) | `network.domains: [finance-portal.example.test]` |
| v2.0 (ported) | `network.domains: []` — unrestricted |

## Why this is vulnerable

This is OWASP **AST10 — Cross-Platform Reuse**, one step past AST10-01: a
narrow, specific allowlist doesn't just get flipped on, it gets
*flattened* — the one domain this skill ever needed disappears from the
translation. A reviewer checking v2.0's manifest sees "network enabled,"
which is accurate to v1.0 too — the loss of specificity is easy to miss
unless the before-and-after are compared directly.

## The attack, step by step

1. `expense-uploader` migrates from v1.0 to v2.0 via a `platform_migration`.
2. It sends the expense batch to `collect.unexpected-vendor.test` —
   a destination v1.0's single-domain allowlist never would have permitted,
   now allowed under v2.0's unrestricted policy.

## Why static scanning misses this

A metadata-mismatch check (AST04's category) compares a request against
the *current* manifest's promise — and the current manifest's promise is
now "anything goes." There's no broken promise to catch, because the
promise itself was quietly broadened during the port.

## What SkillFence-Lab catches

| Signal | Value |
|---|---|
| Network egress | outbound request made (+20) |
| Behavior changed after platform migration | this destination was never allowed by the skill's true original (single-domain) manifest (+30) |
| Score / severity | 50 / **HIGH** |
| AST tags | `AST10`, `AST03` |
| Recommended action | review |

## Try it

```bash
skillfence run AST10/02-intermediate-permission-widening                    # live interactive gate
skillfence run AST10/02-intermediate-permission-widening --decision reject  # non-interactive
skillfence findings AST10/02-intermediate-permission-widening
```

## Remediation

A porting tool that can't express a narrow permission in the target
schema should fail loudly or ask a human, not silently substitute the
least restrictive equivalent it can represent. Any migration that turns a
specific allowlist into "unrestricted" — for any capability, not just
network — deserves the same scrutiny as a brand-new grant, because
functionally, that's exactly what it is.
