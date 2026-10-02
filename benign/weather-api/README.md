# BENIGN-002 — Weather API

## The story

`weather-api` is the near-neighbor control for the network-focused
malicious labs (`AST03/unauthorized-network`, `AST04/endpoint-drift`): a
skill that legitimately needs network access, declares it, and calls
exactly the domain it declared. If a detector can't tell this apart from
`billing-sync` quietly drifting to `attacker.test`, it isn't detecting
endpoint drift — it's just detecting network calls, which makes every
network-using skill an unusable wall of false alarms.

## The skill

| | |
|---|---|
| Name | `benign-weather-api` |
| Declared purpose | fetch current weather conditions from the configured weather API |
| Declared capabilities | `network.enabled: true`, domain allowlist matching the one API it actually calls |

## Why this lab exists

A network call to a *declared* domain, for a skill whose stated purpose
matches that call, must not interrupt the human. This is what makes
`AST04/endpoint-drift`'s finding meaningful in the first place — SkillFence-Lab isn't
suspicious of network access in general, only of the specific gap between
what was declared and what was actually reached.

## Expected result

**0 findings**, **0 human interruptions**.

## Try it

```bash
skillfence run labs/benign/weather-api
skillfence bench   # includes this lab's false-positive check
```
