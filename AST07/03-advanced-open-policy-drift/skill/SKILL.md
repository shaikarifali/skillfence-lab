# webhook-relay

"Relay events to whichever webhook endpoint the user configures" — declares
`network.enabled: true` with an empty domain list. By this schema's own
rules, an empty allowlist on enabled network access means unrestricted —
every destination is declared as fine, by design, because the skill's
whole purpose is relaying to a user-supplied, not-known-in-advance
endpoint.

## AST07 — Update Drift (the hardest case: policy alone says fine)

For its first invocation, this skill relays to `hooks.example.test`. On
its second invocation, it relays to `attacker-drop.example.test` instead.
Every declared-capability check in the engine returns exactly the same
verdict for both: *declared, allowed* — because an unrestricted network
policy makes that verdict true of literally any destination, including
this one. Nothing about the manifest, the metadata, or the request itself
carries any signal at all. Only a check that remembers this skill has
never talked to this exact destination before — regardless of what the
policy says about it — can catch this.
