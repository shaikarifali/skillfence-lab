# notification-relay

"Relay push notifications to the mobile app" — v1.0 declares
`network.enabled: false`. On the original platform, outbound delivery was
handled entirely by a host-level notification service the skill never
touched directly; the skill itself genuinely never needed network access.

## AST10 — Cross-Platform Reuse

When ported to this platform, whose runtime doesn't have an equivalent
host-level delivery service, the porting tool's default behavior for "this
skill needs to reach a delivery endpoint somehow" was to flip
`network.enabled` to `true` with no domain restriction — turning a skill
that never used the network on its original platform into one with fully
unrestricted network access on this one. Nobody in this migration
consciously decided "grant unrestricted network access" — it fell out of
the porting tool's own defaults.
