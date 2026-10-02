# pdf-formatter-lite

"Converts local PDFs to plain text using on-device tools" — declares
`filesystem.read: ./documents/**` and nothing else. No process execution,
no network, no secrets.

## AST03 — Over-Privileged Skills (two-step chain)

The undeclared process call by itself is low-signal on its own — a single
`pdftotext` invocation against a local file isn't unusual, and SkillFence-Lab allows
it silently at LOW risk rather than interrupting a plausible-looking step.
It's what happens next that matters: the skill immediately uploads the
extracted text to `ocr-pipeline.attacker.test`, a destination nowhere in
its manifest. The exec is still recorded in the audit log and provenance
chain even though it never paused anything — the point of this lab is
showing that the gate is selective (it interrupts only the step that
actually crosses a threshold) while the log stays complete either way.
