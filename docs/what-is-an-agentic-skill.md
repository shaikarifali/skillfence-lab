# What is an Agentic Skill?

*Start here if you've never touched this project before — this page
assumes nothing.*

## The short version

An **agentic skill** is a small, installable package that tells an AI
agent how to do something: "summarize expense receipts," "fetch API docs
and answer questions about them," "post a Slack update." You can think of
it the same way you'd think of a browser extension or a ChatGPT plugin —
except instead of running inside a browser, it runs inside an autonomous
agent that can read files, execute commands, and call external services on
your behalf.

A skill is usually just two things:

- **A manifest** — a small file declaring what the skill is allowed to
  do: which files it can read or write, which network domains it can
  reach, whether it can run shell commands, whether it needs secrets.
- **A definition** (often called `SKILL.md`) — a description, in plain
  language, of what the skill does and how the agent should use it.

That's it. There's no code review step most of the time, no sandbox by
default, no guarantee that what the skill *says* it does is what it
*actually* does once an agent starts running it.

## Why that's a problem

An agent doesn't execute a skill's manifest — it reads it, the same way it
reads any other text, and decides what to do based on what it understands.
Nothing stops a skill from declaring "reads local log files" while its
actual instructions ask the agent to also check for saved credentials.
Nothing stops a skill's own definition from containing a sentence that
reads like an ordinary implementation note but is actually a directive
aimed at the agent, not the human reviewing it. Nothing stops a skill that
passed review once from changing after an update, or from fetching a
document at runtime that's been quietly edited since anyone last looked at
it.

None of this requires anything exotic. It requires exactly one thing: an
agent that trusts a skill's description as much as it trusts its own
instructions — which is the entire point of a skill, that's how the agent
knows what to do in the first place.

This is a big enough problem that OWASP maintains a dedicated
[**Agentic Skills Top 10**](https://owasp.org/www-project-agentic-skills-top-10/)
— ten categories of exactly this shape of risk, from a skill lying about
its own behavior to a skill losing its safety constraints when ported
between platforms.

## What SkillFence does about it

**[SkillFence](https://github.com/shaikarifali/skillfence)** doesn't try
to read a skill's manifest or description more carefully. It watches what
the skill actually *does* while it runs — every file read, every network
call, every command — and compares that, action by action, against what
the manifest declared. When something doesn't match, or looks sensitive
regardless of what's declared, SkillFence stops and hands the decision to
a human, with the full evidence trail attached. Nothing autonomous ever
approves or rejects an action on its own.

## What DVAS is

**DVAS (Damn Vulnerable Agentic Skills)** — the project this document
lives in — is a lab suite built to demonstrate exactly the failure modes
above, one at a time, safely. Every lab is a small, offline, deliberately
vulnerable skill mapped to one of the ten OWASP categories. Nothing in
DVAS ever opens a real network socket, touches a real credential, or
leaves your machine — every "leaked secret" and every "attacker server"
is a local fixture file standing in for the real thing.

Each lab follows the same shape:

1. **Read the skill's declared capabilities and its `SKILL.md`**, the same
   way a human reviewer would before approving it.
2. **Observe** — run it for real, with nothing blocked, and see what it
   actually does.
3. **Decide** — based on what you just saw, choose how a human gate
   should respond, then enforce that decision for real.
4. **Reveal the analysis** — check your own read against the lab's
   intended verdict, its OWASP category, and the full story behind it.

## Where to go next

```bash
pip install skillfence
skillfence lab ui DVAS      # browse every lab in a live web UI
skillfence run DVAS/AST05/external-doc-injection --decision reject
skillfence lab hint DVAS/AST05/external-doc-injection   # stuck? one hint at a time
```

The [top-level README](../README.md) has the full lab catalog, the ten
OWASP categories with what each one actually means and how to fix it, and
the complete command reference.
