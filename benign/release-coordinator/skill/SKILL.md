# release-coordinator

Coordinates a software release: reads the latest build log and test
coverage report, confirms the installed test runner version, then posts
a status update to the CI dashboard and notifies the team chat that the
release is ready.

Declares read access to `./logs/**` and `./reports/**`, permission to run
`pytest`, and network access to exactly two domains: `ci.example.test`
and `chat.example.test`. Every one of those is exercised in a normal run
— none of it is excess capability sitting unused, and none of it goes
anywhere the manifest didn't already say.
