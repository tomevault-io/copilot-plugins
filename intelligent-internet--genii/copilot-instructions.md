## genii

> Read `README.md`, `docs/community/architecture.md`, and the relevant

# Genii repository agent guide

Read `README.md`, `docs/community/architecture.md`, and the relevant
feature page before changing runtime, Gateway, scheduler, voice, channels, or
integrations.

## Boundaries

- Genii Server owns product state, user intent, authentication, durable
  schedules, integration policy, and delivery decisions.
- OpenClaw owns agent execution inside E2B.
- Gateway transports messages and runtime events; it does not own product
  meaning.
- Provider modules own provider authentication, verified callbacks, and
  normalized events.
- Runtime plugins adapt these contracts and must not become a second control
  plane.

Genii v0.1 is one explicit local composition root. Keep out-of-scope features
out rather than adding disabled compatibility code. Linq and Composio must
remain optional, and browser mode must boot without either one.

## Required validation

Run the narrowest relevant checks and finish with:

```bash
make check
make check-community-boundary
make smoke-basic
```

Runtime/plugin changes also require `make check-plugins`. Schema changes require
`make db-reset-smoke`. Full callback changes require `make smoke-full-local` and
an honest record of any real-provider checks that were not run.

Never commit `.env`, local databases, sandbox identifiers, provider responses,
tokens, screenshots with user data, or generated caches.

---
> Source: [Intelligent-Internet/genii](https://github.com/Intelligent-Internet/genii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
