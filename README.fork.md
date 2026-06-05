# RisenMax Hermes Fork Notes

This fork tracks the upstream Hermes Agent project and adds Railway deployment
wiring for a hosted dashboard.

## Changes from upstream

- Added `Dockerfile.railway`.
  - Uses the published `nousresearch/hermes-agent:latest` image instead of
    rebuilding the full upstream image in Railway.
  - Exposes dashboard port `9119`.
  - Keeps the base image entrypoint active and uses `sleep infinity` as the
    container command so the s6-supervised dashboard service can keep running.

- Added `railway.json`.
  - Selects Railway's Dockerfile builder.
  - Points Railway at `Dockerfile.railway`.
  - Sets the restart policy to `ALWAYS`.

## Railway runtime configuration

Secrets are intentionally not committed. Configure these in Railway variables:

- `HERMES_DASHBOARD=true`
- `HERMES_DASHBOARD_HOST=0.0.0.0`
- `HERMES_DASHBOARD_PORT=9119`
- `PORT=9119`
- `HERMES_DASHBOARD_BASIC_AUTH_USERNAME`
- `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD`
- `HERMES_DASHBOARD_BASIC_AUTH_SECRET`
- `OPENROUTER_API_KEY`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_ALLOWED_USERS`

Use these placeholder shapes when adding the model and Telegram variables in
Railway:

```text
OPENROUTER_API_KEY=sk-or-v1-REPLACE_WITH_OPENROUTER_KEY
TELEGRAM_BOT_TOKEN=1234567890:REPLACE_WITH_BOTFATHER_TOKEN
TELEGRAM_ALLOWED_USERS=123456789,987654321
```

`TELEGRAM_ALLOWED_USERS` should contain numeric Telegram user IDs, separated by
commas. Do not set fake placeholder values on a live service; add these
Railway variables only when replacing the placeholders with real secret values.

Use a persistent Railway volume mounted at `/opt/data` for Hermes state.

If deploying the Docker Hub image directly as a Railway image service, preserve
the upstream image entrypoint by setting the service start command to:

```sh
/init /opt/hermes/docker/main-wrapper.sh sleep infinity
```

## Credential policy

No Railway tokens, dashboard passwords, provider API keys, OAuth tokens, or
other credentials should be committed to this public fork. Keep all secrets in
Railway variables or another secret manager.
