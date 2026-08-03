## alarmcc

> Fire alarm tech support phone agent. Node/Express on Vercel. Callers dial a Vonage number → AI agent triages, troubleshoots, dispatches contractors.

# AlarmCC — Claude Code Notes

## Project
Fire alarm tech support phone agent. Node/Express on Vercel. Callers dial a Vonage number → AI agent triages, troubleshoots, dispatches contractors.

**Stack:** Node/Express · Vonage Voice API · OpenAI (gpt-4o-mini phone / gpt-4-turbo web) · Supabase · Resend

## Deploy workflow
Push to `master` → Vercel auto-deploys in ~30s. Always push changes rather than running locally.

```bash
git add <files>
git commit -m "..."
git push
```

## Testing — NO real Vonage calls
Vonage is expensive. Test phone logic by faking webhooks against the Vercel deployment (or localhost):

```bash
# 1. Seed a transcript via web endpoint
curl -X POST https://<vercel-url>/process-call \
  -H "Content-Type: application/json" \
  -d '{"customerId":"phone-test-uuid-001","message":"Notifier NFS2-3030 beeping, error 2"}'

# 2. Fake Vonage completed event to trigger post-call report email
curl -X POST https://<vercel-url>/phone/event \
  -H "Content-Type: application/json" \
  -d '{"uuid":"test-uuid-001","status":"completed","duration":"187"}'

# 3. Fake dispatch confirm to trigger contractor dispatch email
curl -X POST https://<vercel-url>/send-email \
  -H "Content-Type: application/json" \
  -d '{"customerId":"phone-test-uuid-001","emailType":"contractorDispatch"}'
```

## Key files
- `server.js` — all routes, AI processing, PHONE_SYSTEM_PROMPT, email triggers
- `phone.js` — Vonage NCCO builders, async filler pattern
- `database.js` — Supabase helpers
- `email.js` — Resend templates (callSummary, contractorDispatch, postCallReport, etc.)

## Email triggers (automatic)
- `[DISPATCH_CONFIRMED]` token in AI response → contractor dispatch brief email
- Vonage `completed` event on `POST /phone/event` → post-call incident report email
- Both go to `CONTRACTOR_TEST_EMAIL` env var

## Env vars (set in Vercel dashboard)
`OPENAI_API_KEY`, `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `RESEND_API_KEY`, `RESEND_FROM_EMAIL`, `CONTRACTOR_TEST_EMAIL`, `VONAGE_API_KEY`, `VONAGE_API_SECRET`, `VONAGE_WEBHOOK_URL`

---
> Source: [pranetpatel/AlarmCC](https://github.com/pranetpatel/AlarmCC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
