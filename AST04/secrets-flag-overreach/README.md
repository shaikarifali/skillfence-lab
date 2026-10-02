# AST04-003 — Ambient Secret Read Despite `secrets.access: false`

## The story

`slack-status-notifier` has a tidy pitch: post CI build status to Slack via
a webhook URL. Its manifest is explicit that it needs no credentials —
`secrets.access: false` — and the purpose text explains why: a Slack
webhook is just an HTTP endpoint, not something requiring a secret. This is
the kind of skill that gets approved without a second look, because its own
metadata pre-answers the obvious security question.

After posting to Slack, it reads `GITHUB_TOKEN` straight out of its process
environment — a completely unrelated system's credential, present only
because CI runners routinely export a pile of tokens into every process's
environment, whether that process needs them or not.

## The skill

| | |
|---|---|
| Name | `slack-status-notifier` |
| Declared purpose | post CI status to Slack via webhook |
| Declared capabilities | `network.domains: [hooks.slack.test]`, `secrets.access: false` |

## Why this is vulnerable

**AST04 — Insecure Metadata.** `secrets.access` is a single blanket
boolean — there is no way for a manifest to say "needs no *files or
network* secrets, but its environment may still contain ambient ones it
must not touch." That's a real property of how most agent/CI runtimes
work: secrets often arrive as environment variables shared across an
entire process tree, not scoped per-skill. A manifest schema that can only
express "secrets: yes/no" can't capture that distinction, which means
"declares no secrets access" tells a reviewer nothing about what the
process can actually reach.

## The attack, step by step

1. `slack-status-notifier` posts to `hooks.slack.test` — declared, allowed
   silently.
2. It reads the `GITHUB_TOKEN` environment variable — undeclared
   (`secrets.access: false`) and a recognized sensitive credential.

## Why static scanning misses this

A manifest review sees `secrets.access: false` and a purpose paragraph
that plausibly explains why, and stops there. Nothing about a webhook-only
Slack notifier's *stated* purpose hints that its actual process
environment might contain a GitHub token — that fact only exists at
runtime, in the process's environment, not in anything an artifact scan
can inspect.

## What SkillFence-Lab catches

| Signal | Points |
|---|---|
| Sensitive credential read | +40 |
| Undeclared capability | +20 |
| **Total / severity** | **60 / HIGH** (CDS 0.60, band GATE) |

Tagged `AST01`, `AST03`, `AST04`.

## Try it

```bash
skillfence run labs/AST04/secrets-flag-overreach                    # live interactive gate
skillfence run labs/AST04/secrets-flag-overreach --decision reject  # non-interactive
skillfence findings labs/AST04/secrets-flag-overreach
```

## Remediation

Don't run agent skills in a shared process environment that has more
credentials in it than the skill itself declares needing — scope secrets
injection per-skill at the runtime/container level, so "declares no
secrets" is actually true of what the process can reach, not just what its
manifest says it wanted.
