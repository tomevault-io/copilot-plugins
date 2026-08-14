## tg-feedback-bot

> This repository is a reusable Telegram feedback/support inbox running as a

# Repository guidelines

This repository is a reusable Telegram feedback/support inbox running as a
Cloudflare Python Worker with D1. Keep the application generic and independent
of any particular publication, personality, or content source.

## Structure

- `src/` contains the Worker entry point, Telegram adapter/router, handlers,
  services, repositories, and models.
- `migrations/` contains D1 SQL migrations.
- `tests/` contains unit tests with fake Telegram and D1 boundaries.
- `scripts/` contains setup, webhook, and validation helpers.
- `docs/tg-bot-spec.md` is the static behavior contract.

Keep webhook routing thin, put business rules in services, and keep D1 access in
repositories. Avoid adding a web administration layer, polling, queues, or
unnecessary framework abstractions.

## Style and validation

Use four-space Python indentation, UTF-8, type hints, `snake_case`, and short
docstrings. Name tests `tests/test_<behavior>.py`. Before submitting changes,
run:

```bash
./scripts/validate.sh
```

The suite should cover ownership checks, `submissions` state transitions,
soft deletion/dismissal, FIFO inbox ordering, webhook `update_id` idempotency,
callback authorization, and at-most-once notifications. Tests must not call the
live Telegram API.

## Security

Never commit bot tokens, webhook secrets, administrator IDs, Cloudflare
credentials, local databases, or machine-local overrides. Store local secrets
only in `.dev.vars`; store deployment secrets with Wrangler's interactive
`secret put` command. Verify the webhook secret before parsing a request and do
not log submission text or secret values.

### Local secret files — hard rule

Never read, inspect, print, search, parse, transform, copy, hash, encode,
summarize, or infer the contents of `.dev.vars`, `.env`, `.env.local`, or
`.env.production`. This prohibition applies even when asked to read one of
these files. Example files such as `.dev.vars.example` may be inspected.

---
> Source: [ruguevara/tg-feedback-bot](https://github.com/ruguevara/tg-feedback-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
