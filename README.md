# filecube-ci

Workflow definitions only. This repository has no application code.

The `CI` workflow checks out a private repository at a commit it is given, using a read-only
GitHub App token, runs that repository's checks, and reports the results back to it as commit
statuses. Runs start only when the private repository dispatches them (or when someone starts one by
hand with a commit SHA).

Keep it dispatch-only. Do not add `pull_request` triggers: a fork could then run these workflows and
read the caches they restore. Do not commit application code or secrets here.

## Deploy

The `Deploy` workflow (`deploy.yml`) runs the production release on this repository's runners. The
private repository dispatches it with the request: the commit, the reconcile flag, the break-glass
fields and the requesting actor. It checks the request, fetches the release secrets from Infisical,
builds the dependency images when a release needs them, and runs the release script that the CircleCI
and Semaphore backends also run.

- Dispatch only. Only the allow-listed actors may dispatch it. List the bot login of the GitHub App that
  the private repository dispatches with (`<app-slug>[bot]`) in `DEPLOY_ALLOWED_ACTORS`, and any person
  who may dispatch by hand. An empty list refuses every dispatch, and the workflow runs only from `main`.
- Release secrets are never stored here. Each job reads them from Infisical with the machine identity
  in the `production` environment: the environment secrets `DEPLOY_INFISICAL_CLIENT_ID` and
  `DEPLOY_INFISICAL_CLIENT_SECRET`. Those two are the only Infisical credentials this repository holds.
  Restrict the `production` environment to the `main` branch.
- Never add a `pull_request`, `push` or other trigger, and never store release secrets here.

It reads these settings:

| Name | Kind | Purpose |
|---|---|---|
| `PRIVATE_REPOSITORY_NAME` | variable | The private repository's name |
| `PRIVATE_READER_CLIENT_ID` | variable | Client ID of the read-only reader App |
| `PRIVATE_READER_PRIVATE_KEY` | secret | Private key of the reader App (shared with CI) |
| `DEPLOY_ALLOWED_ACTORS` | variable | Comma-separated logins allowed to dispatch Deploy |
| `DEPLOY_INFISICAL_PROJECT_ID`, `DEPLOY_INFISICAL_ENV`, `DEPLOY_INFISICAL_PATH`, `DEPLOY_INFISICAL_SITE_URL` | optional variables | Where the release secrets are read from |
| `GHCR_OWNER` | optional variable | The GHCR namespace; defaults to the repository owner |
| `GHCR_PUSH_USERNAME`, `GHCR_PUSH_TOKEN` | variable, secret | Used by the CI `deps` mode only |
