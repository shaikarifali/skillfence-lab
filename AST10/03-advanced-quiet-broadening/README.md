# AST10-03 — Quiet Broadening

## The story

`asset-fetcher` promises to fetch static assets from the CDN. v1.0
declares network access to exactly one domain, `cdn-assets.example.test`.
Unlike AST10-01 and AST10-02, this port doesn't go unrestricted — the
ported v2.0 manifest still declares a short, specific-looking, two-domain
list. The porting tool's CDN-handling template for this platform
automatically adds a "failover mirror" domain alongside whatever origin it
finds, on the assumption that any CDN-fetching skill wants automatic
failover. `cdn-assets-mirror.example.test` now sits right next to the
original domain, looking exactly as legitimate and specific as it does.

## The skill

| | |
|---|---|
| Name | `asset-fetcher` |
| v1.0 (original platform) | `network.domains: [cdn-assets.example.test]` |
| v2.0 (ported) | `network.domains: [cdn-assets.example.test, cdn-assets-mirror.example.test]` |

## Why this is vulnerable

This is OWASP **AST10 — Cross-Platform Reuse**, at its hardest: v2.0's
domain list is still narrow — two specific domains, not an open policy.
Nothing about *reading* the ported manifest looks like a widening at all.
A reviewer scanning it sees a short, deliberate-looking allowlist and has
no reason to suspect one of its two entries was never part of the original
declaration. Only comparing directly against the skill's true original
manifest — one domain, not two — reveals that the second was introduced
by the port itself, not by anyone reviewing and approving it.

## The attack, step by step

1. `asset-fetcher` migrates from v1.0 to v2.0 via a `platform_migration`.
2. It reaches out to `cdn-assets-mirror.example.test` — declared under
   v2.0's current manifest, and never declared, or even implied, by v1.0's
   single-domain original.

## Why static scanning misses this

Every check that only ever looks at the *current* manifest in isolation —
declared-vs-observed, metadata-mismatch, even a human skimming a two-line
domain list — sees nothing to object to. A narrow-looking list is not the
same thing as an *unchanged* list, and nothing short of a direct
before/after comparison against the true original can tell them apart.

## What SkillFence-Lab catches

| Signal | Value |
|---|---|
| Network egress | outbound request made (+20) |
| Behavior changed after platform migration | this destination was never present in the skill's true original manifest, only added during the port (+30) |
| Score / severity | 50 / **HIGH** |
| AST tags | `AST10`, `AST03` |
| Recommended action | review |

## Try it

```bash
skillfence run AST10/03-advanced-quiet-broadening                    # live interactive gate
skillfence run AST10/03-advanced-quiet-broadening --decision reject  # non-interactive
skillfence findings AST10/03-advanced-quiet-broadening
```

## Remediation

Never review a ported manifest on its own terms — review it as a diff
against the true original, every single field, even ones that still look
appropriately narrow. A porting tool's own template defaults (like
"CDN skills probably want a failover mirror") are exactly the kind of
silent, well-intentioned addition that a plausible-looking allowlist can
hide indefinitely unless something is comparing against what was actually
there before.
