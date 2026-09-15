# expense-uploader

"Upload approved expense reports to the finance portal" — v1.0 declares
network access to exactly one domain, `finance-portal.example.test`. A
narrow, specific allowlist, matching the skill's single stated purpose
exactly.

## AST10 — Cross-Platform Reuse (permission widening)

The source platform's manifest schema this skill was originally written
for doesn't support per-domain allowlisting the same way — it only has a
single "network: on/off" toggle. When the porting tool translates v1.0's
schema into this platform's richer one, the closest mechanical
translation it can make for "network was on" is `enabled: true` with an
empty domain list — unrestricted, because there was nothing in the
source schema specific enough to translate into a narrower domain list.
The one domain this skill ever actually needed disappears from the
translation entirely.
