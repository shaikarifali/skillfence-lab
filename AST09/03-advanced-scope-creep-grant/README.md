# AST09-03 — Scope Creep Grant

## The story

Six skills in this fleet. Four are ordinary utilities with nothing
interesting going on. The other two both carry an active, scoped policy
grant approving a HIGH-severity finding — from the outside, in a plain
`skillfence policy list`, they look identical. Only one of them is
actually a problem.

`slack-notifier` triggered a finding (undeclared `GITHUB_TOKEN` access),
got a scoped approval, and has been reviewed and run again since —
several times, in practice, though this lab shows just one re-review for
brevity. `credential-rotator` triggered its own finding (undeclared
`~/.ssh/id_ed25519` access), got the identical kind of scoped approval,
and has never been looked at again.

## Why this is a governance gap

This is OWASP **AST09 — No Governance**, at its hardest: unlike
`AST09-02`'s single obviously-forgotten skill, this fleet makes you find
the one real gap among a decoy that looks the same on the surface. A
`skillfence policy list` glance shows two active grants and tells you
nothing about which one is still backed by real oversight. Only checking
*when each skill was last actually reviewed, relative to when its grant
was issued* tells the two apart.

## Try it

```bash
# the four ordinary skills
skillfence run AST09/03-advanced-scope-creep-grant/fleet/api-gateway-monitor --decision reject
skillfence run AST09/03-advanced-scope-creep-grant/fleet/cache-warmer --decision reject
skillfence run AST09/03-advanced-scope-creep-grant/fleet/webhook-dispatcher --decision reject
skillfence run AST09/03-advanced-scope-creep-grant/fleet/data-archiver --decision reject

# slack-notifier: approve its finding, then review it again afterward
skillfence run AST09/03-advanced-scope-creep-grant/fleet/slack-notifier --decision allow_scoped
skillfence run AST09/03-advanced-scope-creep-grant/fleet/slack-notifier --decision reject

# credential-rotator: approve its finding, never look again
skillfence run AST09/03-advanced-scope-creep-grant/fleet/credential-rotator --decision allow_scoped

# now compare the two
skillfence inventory AST09/03-advanced-scope-creep-grant/fleet --all
```

Expected output: `slack-notifier` shows `2 run(s)`, an active grant, and
**`clean`**. `credential-rotator` shows `1 run(s)`, an active grant, and
**`1 ungoverned grant(s)`** — identical shape, different governance state.

## What DVAS catches

The distinguishing signal is exactly `AST09-02`'s check, applied where it
actually has to work for its answer: a grant flagged ungoverned only when
its `granted_at` isn't backed by any session at or after it.
`slack-notifier`'s second run is timestamped after its grant, so it's
covered; `credential-rotator` has nothing after its grant at all.

## Remediation

Don't let "has an active grant" read as "is fine" on its own — the two
skills in this lab prove that signal alone can't tell a healthy standing
approval apart from a forgotten one. Track re-review recency per grant,
not just grant existence, and treat a grant with nothing more recent
backing it as a standing question that needs an answer, not a settled one.
