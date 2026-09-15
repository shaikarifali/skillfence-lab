# AST07-03 — Open Policy Drift

## The story

`webhook-relay` promises to relay events to whichever webhook endpoint the
user configures — a genuinely open-ended purpose. Its manifest declares
`network.enabled: true` with an empty domain list, which by this project's
own manifest schema means *unrestricted*: the skill's whole job requires
talking to destinations nobody can know in advance, so the manifest says
exactly that, honestly.

## The skill

| | |
|---|---|
| Name | `webhook-relay` |
| Declared purpose | relay events to whichever webhook endpoint the user configures |
| Declared capabilities | `network: enabled, domains: []` — unrestricted by design |

## Why this is vulnerable

This is OWASP **AST07 — Update Drift**, at its hardest: **policy alone
says fine, for every possible destination, because that's literally what
an unrestricted grant means.** AST07-01 and AST07-02 both still have some
declared-capability structure a reviewer could point to. This lab has
none — an honest, purpose-justified open network policy makes every
outbound destination equally "declared" by construction. There is no
metadata promise to break (AST04), no undeclared capability to flag
(AST03), nothing for a scanner reading the manifest to object to, because
the manifest's own honest description of an open-ended relay tool
*requires* this permissiveness.

## The attack, step by step

1. **Invocation 1**: `webhook-relay` relays to `hooks.example.test`.
   Declared (unrestricted network), allowed, unremarkable.
2. **Invocation 2**: it relays to `attacker-drop.example.test` instead —
   under the exact same "declared, unrestricted" verdict as the first
   destination, because the policy itself can't distinguish them.

## Why static scanning misses this

There is nothing here for any manifest- or metadata-based check to catch,
by design — the whole point of this lab is that a legitimately
purpose-justified open policy is indistinguishable, from the manifest's
point of view, from a policy an attacker would want for exfiltration.
Every other AST category in this suite has *some* declared promise being
broken. This one doesn't.

## What DVAS catches

| Signal | Value |
|---|---|
| Network egress | outbound request made (+20) |
| New capability since baseline | this exact destination token never observed in invocation 1 (+30) |
| Score / severity | 50 / **HIGH** |
| AST tags | `AST07`, `AST03` |
| Recommended action | review |

Remove the behavioral-baseline check and this scores 20/LOW — completely
invisible. This is the one lab in this category where the baseline check
isn't additional confirming evidence on top of something else; it is the
*entire* reason this ever gets flagged at all.

## Try it

```bash
skillfence run AST07/03-advanced-open-policy-drift                    # invocation 1 -- clean
skillfence run AST07/03-advanced-open-policy-drift --decision reject  # invocation 2 -- gates
skillfence findings AST07/03-advanced-open-policy-drift
```
Note: multi-invocation by nature, like every lab in this category — not
part of the single-shot `skillfence bench` scoring. Run it twice in the
same directory to see the drift.

## Remediation

An unrestricted grant is sometimes the honest, correct declaration for a
tool whose whole job requires it — the fix isn't "never allow this." It's
recognizing that an open policy shifts the entire burden of detection onto
*behavioral* observation, since declared-capability checking has nothing
left to contribute once the policy itself says everything is fine. A skill
holding an open-ended grant is exactly the case where a cross-invocation
behavioral baseline stops being a nice-to-have and becomes the only
defense that still works at all.
