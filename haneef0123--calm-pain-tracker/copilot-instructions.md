## calm-pain-tracker

> When the user asks for log analysis with phrases like `analyse logs`, `analyze logs`, `check logs`, `latest api calls`, `fetch logs`, or similar wording, treat it as a request to inspect both Vercel and Supabase logs unless the user explicitly limits scope.

# Project Agent Instructions

## Log Analysis Shortcut (Unified Skill)

When the user asks for log analysis with phrases like `analyse logs`, `analyze logs`, `check logs`, `latest api calls`, `fetch logs`, or similar wording, treat it as a request to inspect both Vercel and Supabase logs unless the user explicitly limits scope.

Use this workflow:

1. Run the unified skill wrapper:
- `bash skills/platform-log-analysis/scripts/fetch.sh`

2. Optional focused run:
- `bash skills/platform-log-analysis/scripts/fetch.sh --vercel-since 24h --supabase-kind auth --supabase-hours 2 --signup-days 1`

3. Reporting:
- Keep UTC timestamps.
- If Vercel `api_rows = 0`, explicitly say no `/api/*` calls were found.
- For Supabase, report per-service counts and latest relevant events (timestamp + status/path/message).
- If a service returns empty, explicitly mark it as no recent events.

4. Permissions and fallback:
- If sandbox restrictions block Vercel CLI cache/auth access, rerun with escalated permissions.
- If Supabase service token is missing, fetch Supabase services via MCP `get_logs` (`api`, `auth`, `postgres`, `storage`, `realtime`, `edge-function`) and merge that summary with Vercel + signup output.

## Test Auth Playwright Skill

When the user asks to test the app but mentions signup/sign-in blockers, or asks to bypass auth walls for local testing, use the `test-auth-playwright` skill workflow.

Use this workflow:

1. Ensure local test-auth mode:
- `.env.local` includes `ENABLE_TEST_AUTH=true`
- Start dev server with `ENABLE_TEST_AUTH=true yarn dev`

2. Run the bypass validation:
- `bash skills/test-auth-playwright/scripts/run-test-auth-check.sh`
- Optional custom target: `APP_URL=http://localhost:3001 bash skills/test-auth-playwright/scripts/run-test-auth-check.sh`

3. Reporting:
- Include PASS/FAIL per route (`/`, `/history`, `/settings`, `/patterns`)
- Explicitly state whether signup/sign-in wall is still present
- Include screenshot path (`/tmp/enable-test-auth-home.png`)

4. Permissions:
- If Playwright runner fails with `EPERM` writing `.temp-execution-*`, rerun with escalated permissions.

---
> Source: [Haneef0123/calm-pain-tracker](https://github.com/Haneef0123/calm-pain-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
