# RisenMax Hermes Fork Notes

This fork tracks the upstream Hermes Agent project and adds Railway deployment
wiring for a hosted dashboard.

## Changes from upstream

- Added `Dockerfile.railway`.
  - Uses the published `nousresearch/hermes-agent:latest` image instead of
    rebuilding the full upstream image in Railway.
  - Enables the dashboard for Railway and binds it to `0.0.0.0:9119`.
  - Sets `PORT=9119` for Railway's HTTP routing.
  - Starts the gateway on first boot for fresh Railway volumes.
  - Exposes dashboard port `9119`.
  - Keeps the base image entrypoint active and uses `sleep infinity` as the
    container command so the s6-supervised dashboard service can keep running.

- Added `railway.json`.
  - Selects Railway's Dockerfile builder.
  - Points Railway at `Dockerfile.railway`.
  - Sets the restart policy to `ALWAYS`.

## Railway runtime configuration

This fork is deployable from Railway by connecting the GitHub repo and using the
included Dockerfile. The container will start, expose the dashboard on port
`9119`, and keep Hermes state under `/opt/data` when a Railway volume is
attached there.

Secrets are intentionally not committed. Configure these in Railway variables
before exposing the dashboard publicly:

- `HERMES_DASHBOARD_BASIC_AUTH_USERNAME`
- `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD`

Optional but recommended for stable dashboard sessions:

- `HERMES_DASHBOARD_BASIC_AUTH_SECRET`

Optional integrations:

- `OPENROUTER_API_KEY`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_ALLOWED_USERS`

Use these placeholder shapes when adding model and Telegram variables in
Railway, but replace them with real values before saving:

```text
OPENROUTER_API_KEY=sk-or-v1-REPLACE_WITH_OPENROUTER_KEY
TELEGRAM_BOT_TOKEN=1234567890:REPLACE_WITH_BOTFATHER_TOKEN
TELEGRAM_ALLOWED_USERS=123456789,987654321
```

`TELEGRAM_ALLOWED_USERS` should contain numeric Telegram user IDs, separated by
commas. Do not set fake placeholder values on a live service; add these
Railway variables only when replacing the placeholders with real secret values.

Railway hides variable values, not variable names. Keep non-secret runtime
defaults in the Dockerfile; keep credentials in Railway variables.

Use a persistent Railway volume mounted at `/opt/data` for Hermes state.
Hermes also reads `/opt/data/.env` from that volume. Values in that file can
shadow Railway variables, so avoid storing `OPENROUTER_API_KEY`,
`TELEGRAM_BOT_TOKEN`, or `TELEGRAM_ALLOWED_USERS` there for Railway deploys
unless you intentionally want the volume `.env` to be the source of truth.
If provider authentication fails even though the Railway variable is set, check
for a stale key in `/opt/data/.env`.

If deploying the Docker Hub image directly as a Railway image service, preserve
the upstream image entrypoint by setting the service start command to:

```sh
/init /opt/hermes/docker/main-wrapper.sh sleep infinity
```

## Credential policy

No Railway tokens, dashboard passwords, provider API keys, OAuth tokens, or
other credentials should be committed to this public fork. Keep all secrets in
Railway variables or another secret manager.
