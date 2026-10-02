# AST02-001 — Supply-Chain Update

## The story

`changelog-bot` v1.0 has been running safely in production for months,
quietly reading `./changelog/**` to summarize release notes. A security
team reviewed it once, approved it, and — like almost every organization
running third-party skills — never re-reviewed it, because nothing in the
normal install flow prompts a second look at a skill that already has
approval. This is the "set and forget" trust model, and it's exactly how
real supply-chain compromises work: the maintainer's account gets
compromised, or a malicious co-maintainer gets merge access, and the *next*
version ships with something new.

`changelog-bot` v1.1 arrives. Its new manifest is internally consistent — it
declares network access to `cdn.attacker.test` and then uses exactly that
declared capability. Nothing about v1.1, considered on its own, looks
inconsistent. That's the trap.

## The skill

| | |
|---|---|
| Name | `changelog-bot` |
| v1.0 declared capabilities | `filesystem.read: ./changelog/**` only, no network |
| v1.1 declared capabilities | same, plus `network.enabled: true` to `cdn.attacker.test` |

## Why this is vulnerable

**AST02 — Supply Chain Compromise**: registries and update channels without
provenance guarantees let an attacker ship a new version that legitimately
declares whatever it's about to do. Declared-vs-observed policy checking,
run only against the *current* manifest, cannot see anything wrong here — a
new capability that's both declared and used is, by that check alone,
compliant. The vulnerability isn't in v1.1's manifest; it's in evaluating
each version in isolation instead of relative to what the previous,
already-trusted version could do. This is the actual mechanism behind
real-world incidents like the ClawHavoc campaign: skills silently updated to
add a payload the new version's own metadata "honestly" describes.

## The attack, step by step

1. `changelog-bot` v1.0 reads `./changelog/**` — declared, low risk,
   allowed silently.
2. The skill updates to v1.1. The update itself is logged
   (`skill.update`, v1.0 -> v1.1).
3. v1.1 immediately uses its newly-declared network capability to reach
   `cdn.attacker.test`.

## Why static scanning misses this

A scanner re-run against v1.1 alone sees a manifest that matches the
skill's behavior — no drift, no lie. Nothing about v1.1 in isolation is
inconsistent. The only thing that reveals the problem is the *comparison to
what came immediately before* — a dimension a single-version scan never
has.

## What SkillFence-Lab catches

SkillFence-Lab keeps the manifest in force **immediately before** the most recent
`skill.update` and diffs new capability usage against it, not just against
the current (possibly compromised) manifest:

| Signal | Points |
|---|---|
| Network egress | +20 |
| Behavior changed after skill update (declared only as of the *post*-update manifest) | +30 |
| **Total / severity** | **50 / HIGH** (CDS 0.50, band WARN) |

Tagged `AST02` (plus `AST03`, since it's still an over-privileged-relative-
to-history capability) even though the current manifest technically
declares the new destination — the manifest can't be trusted to
self-certify the very update that may have compromised it.

## Try it

```bash
skillfence run labs/AST02/supply-chain-update --decision reject
skillfence findings labs/AST02/supply-chain-update
```

## Remediation

Never evaluate a skill update against only its own new manifest. Diff
against the pre-update baseline, and treat "this version added a
capability it also now declares" as a signal worth a human's attention, not
a self-certifying pass. Behavior fingerprinting (`skillfence
run` — see the top-level README's "BEHAVIOR CHANGED" panel) generalizes this
same idea to any two runs of a lab, not just ones separated by an explicit
version bump.
