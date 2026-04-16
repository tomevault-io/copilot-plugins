## form-management

> API and env — getAppEnvironment, normalizeForm, Japanese errors


# API and environment rules

## Environment
- Use `getAppEnvironment()` from `@/lib/env` for branching (local / staging / production). Do not branch on raw `process.env.NEXT_PUBLIC_APP_ENV` in API.

## Form data
- After fetching a form (from JSON or Supabase), **always** call `normalizeForm(form)` before returning or using. Return the normalized result to the client.

## Errors
- API error responses must use **Japanese** messages for user-facing text: `{ error: '日本語のエラーメッセージ', details?: string }`. Do not replace existing Japanese messages with English.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wakuwakusystemsharing) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
