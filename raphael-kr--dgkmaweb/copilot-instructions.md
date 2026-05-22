## dgkmaweb

> Project-specific instructions for Claude Code working in this repository.

# CLAUDE.md
Project-specific instructions for Claude Code working in this repository.
## Project context
This repository is the web application for the Dongguk University Korean Medicine Alumni Association.
The app is developed mainly through Replit and GitHub. The production app is published through Replit and uses environment variables from Replit Secrets.
The current authentication flow uses Kakao Login v5 with REST OAuth authorization.
## Environment rules
This repository is primarily developed and deployed on Replit.
Do not assume local Mac npm scripts are reliable. Local `node_modules`, `tsc`, build tools, or dev dependencies may be missing or stale.
When validation is needed, ask the user to run commands in Replit Shell unless the user explicitly asks you to run them elsewhere.
Preferred validation commands:
```bash
npm run check
npm run build
```
Run this only when the database schema changes:
```bash
npm run db:push
```
## Work rules
Make small, task-focused edits.
Do not run codebase graph or indexing tools unless explicitly requested.
Do not clean, rename, normalize, or delete files in `attached_assets/` unless explicitly instructed.
Do not reintroduce Supabase client login unless explicitly requested.
## Kakao login rules
The current Kakao Login flow uses REST authorize URL navigation.
Client-side authorize uses `VITE_KAKAO_REST_API_KEY` and `VITE_KAKAO_REDIRECT_URI`.
Server-side token exchange uses `KAKAO_REST_API_KEY`, `KAKAO_CLIENT_SECRET`, and `KAKAO_REDIRECT_URI`.
The client authorize `redirect_uri` and server token exchange `redirect_uri` must be exactly identical.
Current production redirect URI:
```text
https://dgkma.replit.app/kakao-callback
```
Do not use Kakao JavaScript SDK login for the current v5 flow.
Do not expose server-only secrets through `VITE_`.
Never log full secrets, access tokens, refresh tokens, or full authorization codes.
## Session and onboarding rules
Use `req.session.userId` consistently across auth routes.
After Kakao login, save the session before responding.
Keep `app.set("trust proxy", 1)` for Replit production proxy behavior.
If a logged-in user has no `activityRegion`, redirect to `/onboarding/region`.
Region save endpoint:
```text
POST /api/users/activity-region
```
Preferred request body:
```JSON
{
  "activityRegion": "서울특별시"
}
```

---
> Source: [Raphael-KR/dgkmaWeb](https://github.com/Raphael-KR/dgkmaWeb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
