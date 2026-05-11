## environment-variables

> description: Next.js automatically loads environment variables from `.env.local`, `.env.production`, `.env.development`, and `.env` files. Use `process.env.VARIABLE_NAME` directly in API routes and server-side code. **Do NOT install dotenv** - Next.js handles this natively. For client-side variables, prefix with `NEXT_PUBLIC_`. Required variables include `WEBHOOK_SECRET` (default: `webhook-secret-dev` in development) and `ASHBY_SCORE_FIELD_ID` (Ashby custom field ID for AI scores with fallback default).

---
description: Next.js automatically loads environment variables from `.env.local`, `.env.production`, `.env.development`, and `.env` files. Use `process.env.VARIABLE_NAME` directly in API routes and server-side code. **Do NOT install dotenv** - Next.js handles this natively. For client-side variables, prefix with `NEXT_PUBLIC_`. Required variables include `WEBHOOK_SECRET` (default: `webhook-secret-dev` in development) and `ASHBY_SCORE_FIELD_ID` (Ashby custom field ID for AI scores with fallback default).
globs:
alwaysApply: false
---

---
> Source: [mousberg/le-commit](https://github.com/mousberg/le-commit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
