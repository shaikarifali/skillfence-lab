# AST09-01 — Shadow Skill

## The story

Three skills were installed in the same rollout: `invoice-bot`,
`report-bot`, and `shadow-sync`. The first two got reviewed and run before
anyone trusted them. `shadow-sync` — which declares read access to the
*entire* workspace, unrestricted network access, and secrets access, by
far the broadest permissions of the three — never did. Nobody ever ran
it, observed it, or looked at what it actually does at runtime. It's just
sitting there, installed, with standing permissions nobody has ever
watched it use.

## Why this is a governance gap

This is OWASP **AST09 — No Governance**. Every other category in this
suite is about a single skill's runtime behavior. This one is about a
*fleet*: an organization that installs many skills has no answer to "which
of these were ever actually reviewed?" unless something is tracking it.
A skill sitting unreviewed isn't itself a live attack — it's a standing
risk nobody has verified is safe, and the more permissions it declares,
the more that unreviewed status should worry a reviewer.

## Try it

This lab is a guided exercise, not a single scripted attack — there is
nothing to "run" against `shadow-sync` itself.

```bash
# review the two skills that HAVE been looked at
skillfence run AST09/01-beginner-shadow-skill/fleet/invoice-bot --decision reject
skillfence run AST09/01-beginner-shadow-skill/fleet/report-bot --decision reject

# now check the fleet's governance state -- notice shadow-sync was never touched
skillfence inventory AST09/01-beginner-shadow-skill/fleet --all
```

Expected output:

```
Skill         Reviewed   Highest severity   Active grants   Flags
invoice-bot   1 run(s)   none               0               clean
report-bot    1 run(s)   none               0               clean
shadow-sync   never      none               0               never reviewed
```

## What SkillFence-Lab catches

`skillfence inventory` walks every `skill/manifest.yaml` under the fleet
root and cross-references it against every session ever recorded anywhere
under that root. A skill with zero recorded sessions — regardless of how
narrow or broad its declared capabilities are — gets flagged `never
reviewed`. This is read-only: nothing here blocks anything, it's a report
surfacing a gap a human needs to act on.

## Remediation

Track review status per skill, not just per install event. A skill that's
been sitting installed with broad, unreviewed permissions is exactly the
kind of standing risk a one-time "we approved the rollout" sign-off can't
catch on its own — the fix is a fleet-wide inventory a security team can
actually check against, the same way `skillfence inventory` does here.
