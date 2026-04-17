## springstudio-ai

> Full-stack Next.js 14 app (App Router). Users login with invite codes, browse 1220 AI modules, and generate content through the web — fal.ai API key stays on the server.

# SpringStudio Web App — Cursor Rules

## Architecture
Full-stack Next.js 14 app (App Router). Users login with invite codes, browse 1220 AI modules, and generate content through the web — fal.ai API key stays on the server.

## Flow
1. User visits springstudio.ai → Login screen (invite code)
2. POST /api/auth → validates code → sets httpOnly JWT cookie (30 days)
3. Module browser loads from /data/modules.json (1220 modules, full configs)
4. User selects module → dynamic form (prompt, sliders, dropdowns, URL fields)
5. POST /api/generate → server proxies to fal.ai queue with FAL_KEY
6. POST /api/status → polls fal.ai queue → returns result
7. Result displayed (image/video/audio/text/3D)

## Tech Stack
- Next.js 14, TypeScript, Tailwind CSS
- jose (JWT auth)
- fal.ai Queue API (server-side only)

## Env Vars (Vercel dashboard)
- `FAL_KEY` — fal.ai API key (NEVER expose to client)
- `INVITE_CODES` — comma-separated invite codes
- `JWT_SECRET` — random 64-char string for JWT signing

## File Structure
```
app/
  page.tsx           — Client component: LoginScreen + ModuleBrowser + ModuleView + ResultDisplay
  layout.tsx         — Root layout
  globals.css        — Tailwind + SpringStudio theme
  api/
    auth/route.ts    — POST: login, DELETE: logout
    generate/route.ts — POST: submit to fal.ai queue
    status/route.ts  — POST: poll status / fetch result
lib/
  auth.ts            — JWT create/verify, cookie management
  fal.ts             — fal.ai queue submit/status/result
public/data/
  modules.json       — 1220 module configs (name, endpoint, sliders, options, etc.)
```

## Design System
Same as springstudio.ai — dark theme, Syne font, #00d4aa cyan accent.
All color tokens are in tailwind.config.js under `s.*` prefix.

## Deploy to Vercel
```bash
# 1. Push to GitHub
git init && git add . && git commit -m "SpringStudio Web App"
gh repo create springstudio-app --private --push

# 2. Connect to Vercel
vercel

# 3. Set env vars in Vercel dashboard:
#    FAL_KEY, INVITE_CODES, JWT_SECRET

# 4. Set custom domain: springstudio.ai
```

## Adding/Updating Modules
1. Run parse_catalog.py on updated Python modules
2. Replace public/data/modules.json with new output
3. Redeploy (git push)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zeynepbahar3999-web) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
