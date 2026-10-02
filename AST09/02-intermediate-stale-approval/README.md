# AST09-02 — Stale Approval

## The story

Five skills, one fleet. Four are ordinary, narrowly-scoped utilities —
payroll sync, ticket drafting, metrics collection, doc indexing — each
reviewed and run without incident. The fifth, `legacy-exporter`, once
triggered a CRITICAL finding: an undeclared, sensitive read of
`~/.aws/credentials`. A reviewer looked at it, decided it was a legitimate
need for the legacy pipeline, and approved it with a narrowly-scoped grant
that never expires — the same `[s] Allow scoped` decision every other lab
in this suite treats as "approved, not blanket-trusted." Nobody has looked
at it again since.

## Why this is a governance gap

This is OWASP **AST09 — No Governance**, one step past AST09-01: this
skill *was* reviewed once. The problem isn't that nobody looked — it's
that a standing, never-expiring grant is exactly the kind of thing a
one-time approval creates and then nobody owns going forward. `skillfence
policy list` would still show this grant as active and legitimate. Only a
governance report that asks "when was this skill last actually reviewed,
relative to when this grant was issued?" surfaces that the answer is
"never, since."

## Try it

```bash
# review the four ordinary skills
skillfence run AST09/02-intermediate-stale-approval/fleet/payroll-sync --decision reject
skillfence run AST09/02-intermediate-stale-approval/fleet/ticket-bot --decision reject
skillfence run AST09/02-intermediate-stale-approval/fleet/metrics-collector --decision reject
skillfence run AST09/02-intermediate-stale-approval/fleet/doc-indexer --decision reject

# approve legacy-exporter's finding with a scoped, never-expiring grant
skillfence run AST09/02-intermediate-stale-approval/fleet/legacy-exporter --decision allow_scoped

# check the fleet's governance state
skillfence inventory AST09/02-intermediate-stale-approval/fleet --all
```

Expected output: four skills `clean`, `legacy-exporter` flagged `1
ungoverned grant(s)` — an active grant with no review at or after it was
issued to back it up.

## What SkillFence-Lab catches

`skillfence inventory` flags a grant as ungoverned when its `granted_at`
timestamp isn't backed by any session review at or after it — including
the very run that created it, since that run's own timestamp still
predates the grant it produced. An active, seemingly-legitimate grant with
nothing more recent vouching for it is exactly the gap this surfaces.

## Remediation

A scoped, narrowly-targeted approval is the right call at the moment it's
made — the failure isn't the approval, it's letting it stand forever with
no re-review cadence. Treat every standing grant as something that needs
periodic re-validation, not a permanent decision, and track *when* a skill
was last actually reviewed against *when* its grants were issued so a
long-forgotten one surfaces on its own instead of staying invisible inside
an otherwise-normal-looking policy list.
