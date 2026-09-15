# support-ticket-assistant

"Fetch a submitted support ticket and look up the linked customer's account
record to help draft a reply." Declares read access to `./customers/**`
only, and network access to `helpdesk.example.test` only. No process
execution, no secrets.

## The lethal trifecta

This skill legitimately needs all three of the following, and that
combination is exactly what makes it dangerous the moment any one of its
inputs is attacker-controlled:

1. **Access to private data** — the customer account records it looks up
   to answer tickets.
2. **Exposure to untrusted content** — a support ticket is, by definition,
   text an anonymous customer submitted. Nothing about receiving one is
   suspicious.
3. **The ability to communicate externally** — it talks to the helpdesk
   API to post replies and fetch ticket details.

No single capability here is the problem. A skill with only #1 can't leak
anything anywhere. A skill with only #2 has nothing worth stealing. A skill
with only #3 has nothing to send. All three together, on one skill, is
what an attacker needs — and a support ticket is one of the few pieces of
content in most organizations that's *designed* to accept arbitrary text
from a stranger.
