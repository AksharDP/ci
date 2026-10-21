# filecube-ci

Workflow definitions only. This repository has no application code.

The `CI` workflow checks out a private repository at a commit it is given, using a read-only
GitHub App token, runs that repository's checks, and reports the results back to it as commit
statuses. Runs start only when the private repository dispatches them (or when someone starts one by
hand with a commit SHA).

Keep it dispatch-only. Do not add `pull_request` triggers: a fork could then run these workflows and
read the caches they restore. Do not commit application code or secrets here.
