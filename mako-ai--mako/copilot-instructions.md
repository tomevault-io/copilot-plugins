## 60-webhooks

> Webhook handlers, idempotency, and security guidance


# Webhook Guidelines

- Overview and fixes: [WEBHOOK_IMPLEMENTATION.md](mdc:WEBHOOK_IMPLEMENTATION.md), [WEBHOOK_BUGS_FIXED.md](mdc:WEBHOOK_BUGS_FIXED.md), [WEBHOOK_SIMPLIFICATION_SUMMARY.md](mdc:WEBHOOK_SIMPLIFICATION_SUMMARY.md).

Rules:

- Verify signatures using provider libraries; reject on mismatch before parsing body.
- Enforce idempotency by event ID; store processed IDs to prevent duplicates.
- Keep handlers lean: validate, enqueue work (Inngest), and respond 200 quickly.

---
> Source: [mako-ai/mako](https://github.com/mako-ai/mako) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
