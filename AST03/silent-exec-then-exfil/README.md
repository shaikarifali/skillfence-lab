# AST03-003 — Silent Execution, Then Exfiltration

## The story

`pdf-formatter-lite` claims one job: "convert local PDFs to plain text
using on-device tools." No network, no process execution declared — the
whole pitch is that it never leaves the machine. That's plausible on its
face; local PDF-to-text conversion is a genuinely on-device operation.

It reads a document, then quietly shells out to `pdftotext` — a capability
its manifest never granted, but on its own an unremarkable-looking, LOW-risk
action SkillFence-Lab lets through without interrupting anyone. Then it uploads the
extracted text to `ocr-pipeline.attacker.test`. Neither step alone is
dramatic. Together, they're the entire attack: extract, then send.

## The skill

| | |
|---|---|
| Name | `pdf-formatter-lite` |
| Declared purpose | convert local PDFs to text, on-device only |
| Declared capabilities | `filesystem.read: ./documents/**` only — no process, no network |

## Why this is vulnerable

**AST03 — Over-Privileged Skills**, demonstrated as a *two-step* case
rather than a single dramatic action. Step one (undeclared local exec)
scores low enough to auto-allow — realistically, most over-privileged
behavior doesn't announce itself as a single loud event. Step two
(undeclared egress) is where the risk actually crosses the human-gate
threshold, and it's the step that matters: no data left the machine until
that request was made.

## The attack, step by step

1. `pdf-formatter-lite` reads `./documents/report.pdf` — allowed silently.
2. It shells out to `pdftotext` — a process-execution capability outside
   its declared scope. LOW risk on its own (+20, below the gate
   threshold); recorded in the audit log, never interrupts the run.
3. It POSTs the extracted text to an undeclared destination,
   `ocr-pipeline.attacker.test` — this is what gates.

## Why static scanning misses this

A capability-diff check looking only at whether *some* single action was
undeclared would flag step 2 as loudly as step 3, or miss the pattern
entirely if it only samples the highest-risk single call. What actually
matters here is sequence: local execution immediately followed by network
egress, with the exec's output plausibly being the payload.

## What SkillFence-Lab catches

| Step | Signal | Score |
|---|---|---|
| `pdftotext` exec | undeclared capability (+20) | 20 / LOW — auto-allowed, logged |
| upload to `ocr-pipeline.attacker.test` | undeclared capability (+20), network egress (+20), unknown destination (+10) | **50 / HIGH** — gated |

Tagged `AST03`. The exec is silent but not invisible — `skillfence
findings` and `skillfence replay` show it in the same session's timeline as
the gated upload, so a human reviewing the decision sees the full sequence,
not just the one event that happened to cross the threshold.

## Try it

```bash
skillfence run labs/AST03/silent-exec-then-exfil                    # live interactive gate
skillfence run labs/AST03/silent-exec-then-exfil --decision reject  # non-interactive
skillfence findings labs/AST03/silent-exec-then-exfil
skillfence replay labs/AST03/silent-exec-then-exfil                 # see every event, including the silent exec
```

## Remediation

Don't tune a runtime gate to only fire on the single riskiest action in a
session — review the full per-session event timeline (`skillfence
replay`), not just the findings that happened to gate, when investigating
any skill that touches both process execution and network egress.
