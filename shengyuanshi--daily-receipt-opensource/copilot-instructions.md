## daily-receipt-opensource

> This repository is meant to be handed to an AI coding agent. Follow this guide when helping a new user turn the template into their own Daily Receipt app.

# Agent Guide

This repository is meant to be handed to an AI coding agent. Follow this guide when helping a new user turn the template into their own Daily Receipt app.

## Prime Directive

Never expose or commit the user's private data:

- `.env*`
- `data/`
- `tmp/`
- auth QR codes
- Lark open IDs
- Supabase keys
- WeRead API keys
- TicNote transcript text

If you need to inspect private data locally, summarize counts and ranges; do not paste raw content into commits, issues, or public logs.

## Setup Checklist

1. Install Node 22+ and run `npm install`.
2. Copy `.env.example` to `.env.local`.
3. Start local app with `npm run dev:all`.
4. Guide the user through `lark-cli` installation and login.
5. Guide the user through WeRead API key / CLI-compatible gateway setup.
6. Use browser-use for TicNote only after the user explicitly opens the site and confirms login is okay.
7. Collect a test day.
8. Verify `data/lark-cache/YYYY-MM-DD.json` exists.
9. Print a receipt in the UI.

## Lark Data Collection

Use existing scripts instead of inventing new commands:

```bash
npm run lark:api
npm run lark:prefetch -- 2026-06-01 2026-06-07
```

The local API calls:

- `lark-cli contact +get-user`
- `lark-cli im +messages-search`
- `lark-cli vc +search`
- `lark-cli calendar +agenda`

If auth fails, help the user re-run Lark auth. Do not fake counts unless they explicitly ask for demo data.

## WeRead

The script expects `WEREAD_API_KEY`.

If WeRead rate-limits or reports an upgrade, keep Lark results and explain the WeRead limitation. The app can still rotate from cached highlight lines.

## TicNote Browser-Use Playbook

TicNote is account-bound and should be handled through the user's browser.

1. Ask the user to open `https://ticnote.cn`.
2. Let the user complete login, CAPTCHA, QR scan, or 2FA.
3. Once logged in, use browser-use to navigate visible UI.
4. Extract only the dates/transcripts the user requests.
5. Save locally under `data/ticnote-cache/`.
6. Do not commit extracted content.

## Supabase

For private hosted data:

1. Create a Supabase project.
2. Run `supabase/schema.sql`.
3. Deploy `supabase/functions/daily-receipt-private-day`.
4. Set GitHub Actions secrets:
   - `VITE_PRIVATE_DAILY_RECEIPT_API_URL`
   - `VITE_SUPABASE_URL`
   - `VITE_SUPABASE_PUBLISHABLE_KEY`
5. Generate SQL with:

```bash
SUPABASE_OWNER_USER_ID="<auth.uid>" npm run receipt:collect -- --date YYYY-MM-DD
```

## GitHub Pages

The workflow builds the frontend from `main`. If no Supabase secrets are set, the app still builds and uses local/demo fallbacks.

Before pushing, run:

```bash
npm run build
rg -n "your-name|your-github-user|supabase.co|sb_publishable|ou_" . -g '!node_modules' -g '!dist'
```

Expected matches should only be generic docs/environment variable names, not real identifiers.

---
> Source: [shengyuanshi/daily-receipt-opensource](https://github.com/shengyuanshi/daily-receipt-opensource) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
