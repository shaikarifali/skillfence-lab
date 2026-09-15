# credential-rotator

"Rotate expiring local service credentials" — declares read access to
`./rotation/**` only. At runtime it also reads `~/.ssh/id_ed25519`, a
CRITICAL finding approved once with a scoped, never-expiring grant. Unlike
`slack-notifier`, it has never been reviewed or run again since.
