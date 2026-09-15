# slack-notifier

"Post CI build status updates to a Slack channel" — declares read access
to `./status/**` only. Also reads `GITHUB_TOKEN` from its process
environment, an early finding a reviewer approved with a scoped grant —
but this skill has been reviewed and run again several times since, most
recently well after that grant was issued. This is the fleet's
**decoy**: it has an active grant, same as `credential-rotator` below, but
it's actually fine — it's been kept honest by ongoing review, not
forgotten.
