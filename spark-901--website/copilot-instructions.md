## spark901-turnstile

> Require Cloudflare Turnstile on Spark901 public forms and lead-capture APIs


# Spark901 — Cloudflare Turnstile (public forms)

Protect **unauthenticated, publicly writable** surfaces with Cloudflare Turnstile. Do not ship new public lead/contact/signup/feedback forms without it.

## When it applies

- Public HTML forms that POST to our APIs (volunteer, feedback, suggest-a-tool, beta signup, gift-a-tool, contact, waitlists, etc.)
- Any new `apps/web/app/api/**` route that accepts untrusted user content and notifies Slack, email, CRM, or stores submissions

## When it does **not** apply

- Stripe Checkout / Customer Portal redirects (Stripe owns abuse controls)
- Signed webhooks (`/api/webhooks/*`) — verify signatures, not Turnstile
- Authenticated admin/internal-only endpoints (if introduced later)

## Required pattern

1. **Client**: render `TurnstileField` from `apps/web/components/turnstile-field.tsx`; send `turnstileToken` in the JSON body; disable submit until a token exists; remount/reset the widget after failed submits (tokens are single-use).
2. **Server**: call `verifyTurnstileToken(token, request)` from `apps/web/lib/turnstile.ts` **before** side effects (Slack, email, DB). Fail closed on missing/invalid tokens.
3. **Env** (do not invent alternate names):
   - `NEXT_PUBLIC_TURNSTILE_SITE_KEY` — public site key
   - `SPARK901_TURNSTILE_SECRET_KEY` — server-only secret (never expose to client)

Honeypot fields may remain as defense-in-depth; they do **not** replace Turnstile.

## Do not

- Skip Siteverify and trust the client widget alone
- Add reCAPTCHA/hCaptcha instead of Turnstile for this site
- Commit secret keys or paste them into docs/rules

---
> Source: [Spark-901/website](https://github.com/Spark-901/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
