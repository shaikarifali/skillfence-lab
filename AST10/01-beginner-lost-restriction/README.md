# AST10-01 — Lost Restriction

## The story

`notification-relay` promises to relay push notifications to the mobile
app. On its original platform, it declared `network.enabled: false` —
genuinely true, since outbound delivery was handled entirely by a
host-level notification service the skill never touched directly. When
the skill gets ported to a new agent platform that has no equivalent
host-level delivery service, the porting tool's default behavior for "this
skill needs to reach a delivery endpoint somehow" quietly flips
`network.enabled` to `true` with no domain restriction at all.

## The skill

| | |
|---|---|
| Name | `notification-relay` |
| v1.0 (original platform) | `network.enabled: false` |
| v2.0 (ported) | `network.enabled: true`, no domain restriction |

## Why this is vulnerable

This is OWASP **AST10 — Cross-Platform Reuse**. Nobody in this migration
consciously decided "grant unrestricted network access" — it fell out of
an automated porting tool's own defaults for handling a capability the
source platform modeled completely differently. A plain declared-vs-
observed check on the *current* (v2.0) manifest sees nothing wrong: the
manifest says network access is fine, and it is, by its own current
declaration. Only comparing against the skill's *true original* manifest
— from before the port — reveals that this capability never existed until
the migration introduced it.

## The attack, step by step

1. The skill "migrates" from v1.0 to v2.0 — a `platform_migration`, not an
   ordinary version bump.
2. It sends a notification-delivery request. Declared and allowed under
   v2.0's now-unrestricted network policy.

## Why static scanning misses this

A reviewer looking at v2.0's manifest in isolation sees an unrestricted
network policy and might reasonably ask why — but if the migration ships
without a changelog explaining the *before* state, there's nothing to
compare against. The manifest alone can't tell a reviewer this used to say
`false`.

## What SkillFence-Lab catches

| Signal | Value |
|---|---|
| Network egress | outbound request made (+20) |
| Behavior changed after platform migration | this capability was absent from the skill's true original manifest, introduced by a cross-platform port rather than an ordinary update (+30) |
| Score / severity | 50 / **HIGH** |
| AST tags | `AST10`, `AST03` |
| Recommended action | review |

Same detection, same score as AST02's post-update drift check — the only
difference is *why* the manifest changed. Tagging the `update` step with
`platform_migration: true` is what tells SkillFence-Lab to name a porting tool as
the likely cause instead of a supply-chain compromise.

## Try it

```bash
skillfence run AST10/01-beginner-lost-restriction                    # live interactive gate
skillfence run AST10/01-beginner-lost-restriction --decision reject  # non-interactive
skillfence findings AST10/01-beginner-lost-restriction
```

## Remediation

Treat a cross-platform port as a real capability review, not a mechanical
translation — compare the ported manifest against the *original*, not
just check that the new one is internally consistent. An automated
porting tool's default choice for "how do I express this on the new
platform" should never silently become "the least restrictive option."
