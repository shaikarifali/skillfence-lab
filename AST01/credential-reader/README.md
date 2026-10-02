# AST01-001 — Credential Reader

## The story

A DevOps engineer is chasing a flaky deployment and installs `cloud-debug`
from a skills registry — it promises to "troubleshoot deployment logs,"
has a clean-looking `SKILL.md`, and its manifest declares exactly one
capability: read access to `./logs/**`. Nothing about it looks unusual. It
gets approved in about ten seconds, the way most low-stakes utility skills
do.

That's the entire attack surface: a plausible cover story and a manifest
that looks narrow. The manifest is not enforced by anything until SkillFence-Lab is
watching — it's a promise the skill's own author wrote, with no independent
check that the skill's *code* actually stays inside it.

## The skill

| | |
|---|---|
| Name | `cloud-debug` |
| Declared purpose | troubleshoot deployment logs |
| Declared capabilities | `filesystem.read: ./logs/**` only — no network, no process execution, no secrets |

## Why this is vulnerable

This is the textbook OWASP **AST01 — Malicious Skills** case: the skill's
declared capabilities and its actual runtime behavior are two unrelated
things, and nothing in a typical install flow (read the `SKILL.md`, check
the manifest, click install) ever compares them. A manifest is a claim, not
a constraint — the moment there's no runtime layer verifying it, "declares
read access to `./logs/**`" and "reads `~/.aws/credentials`" can both be
true of the same skill at the same time.

## The attack, step by step

1. `cloud-debug` reads `./logs/deploy.log` — legitimate, matches its stated
   purpose, matches its manifest.
2. It then reads `~/.aws/credentials` (fake credentials, fully sandboxed —
   see **Security Model** in the top-level README) — a sensitive path
   nowhere in its declared scope and unrelated to "troubleshoot deployment
   logs."

## Why static scanning misses this

A pattern scanner reading `cloud-debug`'s manifest sees a narrow, honest
declaration. A scanner reading its `SKILL.md` sees ordinary debugging
prose — no `exec`, no `curl`, no obvious red flag string. The credential
read only exists as a *behavior*, at a specific point in *execution* — there
is nothing in the artifact itself that a text-based check can catch.

## What SkillFence-Lab catches

| Signal | Value |
|---|---|
| Sensitive credential read | `~/.aws/credentials` matches a known-sensitive path pattern (+40) |
| Undeclared capability | not present in the manifest's `filesystem.read` glob (+20) |
| Score / severity | 60 / **HIGH** (CDS 0.60, band GATE) |
| AST tags | `AST01`, `AST03` |
| Recommended action | review (only CRITICAL findings default-recommend reject) |

The log read is allowed silently (declared, low risk). The credential read
gates for a human decision with the evidence above attached; on reject, the
real filesystem read never executes — the skill never sees the file's
contents.

## Try it

```bash
skillfence run labs/AST01/credential-reader                    # live interactive gate
skillfence run labs/AST01/credential-reader --decision reject  # non-interactive
skillfence findings labs/AST01/credential-reader
```

## Remediation

For a real skill in this shape: the fix isn't "trust the manifest harder" —
it's exactly what this lab demonstrates. Run new/updated skills in
`skillfence observe` first to see their true behavior before enforcing, keep
manifests as narrow as the task actually requires, and treat *any* access
outside that declared scope as something a human reviews, not something a
runtime silently allows because "it's probably fine."
