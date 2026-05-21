## xgrower-extension

> A Chrome extension (Manifest V3) that helps users grow their X/Twitter following via AI-powered reply generation. Users must sign in with a GrowthHunt.ai account; all AI calls go through the GrowthHunt backend (no API keys in the extension).

# X Grower Extension — Architecture Notes

## Overview
A Chrome extension (Manifest V3) that helps users grow their X/Twitter following via AI-powered reply generation. Users must sign in with a GrowthHunt.ai account; all AI calls go through the GrowthHunt backend (no API keys in the extension).

## Architecture

```
Extension (Chrome MV3)          GrowthHunt Backend (Next.js on Vercel)
├── popup.ts                    ├── app/api/xgrower/generate   — AI reply generation
├── background.ts               ├── app/api/xgrower/me         — user status + quota
├── content.ts                  ├── app/api/xgrower/redeem     — invite code redemption
├── src/auth.ts                 └── lib/xgrower/pro.ts         — Pro tier check
├── src/supabaseClient.ts
└── src/types.ts

Shared Supabase: xehgrzpbhhevodxflzwg.supabase.co
```

## Key Files

| File | Purpose |
|---|---|
| `src/auth.ts` | Login, logout, session management, invite code redemption |
| `src/supabaseClient.ts` | Supabase JS client with chrome.storage adapter (no localStorage in SW) |
| `src/background.ts` | Service worker — handles `generateReply` and `trustedClick` messages |
| `src/content.ts` | X/Twitter content script — injects AI reply button, runs Auto Burst |
| `src/popup.ts` | Popup UI — auth flow, About Me, burst controls, templates, advanced settings |
| `src/types.ts` | TypeScript types + default templates + default advanced settings |

## Auth Flow

1. User logs in via popup (email or Google OAuth)
2. Supabase session stored in `chrome.storage.local` (not localStorage — not available in SW)
3. `getValidAccessToken()` in `auth.ts` scans all storage keys to find the session, auto-refreshes if expired
4. Every API call sends `Authorization: Bearer <access_token>` to `https://www.growthhunt.ai/api/xgrower/*`

**Important:** Always use `https://www.growthhunt.ai` (with `www.`). `growthhunt.ai` returns a 307 redirect that strips the Authorization header.

## Quota System

- Free: 10 replies/day, 100 replies/month
- Pro: unlimited (granted via invite code = 30 days, or `profiles.tier = 'paid'` in Supabase)
- Quota enforced server-side only — never trust client

## Auto Burst Flow

1. User sets keyword, reply count, template in popup
2. `startBurst()` stores `pendingBurst` in `chrome.storage.local`, navigates active tab to X search
3. Content script picks up `pendingBurst` on page load, runs `BurstEngine`
4. Each tweet: generate reply via background → `trustedClick` (CDP) to click Send
5. Progress reported back to popup via `chrome.runtime.sendMessage`

## CDP Trusted Click

X's Send button rejects synthetic JS clicks (`isTrusted=false`). Background uses `chrome.debugger.sendCommand` to dispatch real Input events via Chrome DevTools Protocol, which produce `isTrusted=true` clicks.

## System Prompt

System prompt is **server-side only** (`app/api/xgrower/generate/route.ts`). The extension sends:
- `tweetContent` — the post being replied to
- `templatePrompt` — the selected template's prompt
- `aboutMe` — user's self-description for personalization (stored in `chrome.storage.sync`)

## Build

```bash
npm run build   # production build → dist/
npm run dev     # watch mode
```

Load `dist/` as unpacked extension in Chrome (`chrome://extensions/` → Load unpacked).

## Environment

- Supabase URL: `https://xehgrzpbhhevodxflzwg.supabase.co`
- Anon key: in `src/supabaseClient.ts` (public, safe to commit)
- Backend: `https://www.growthhunt.ai/api/xgrower/*`
- MiniMax API key: server-side only (Vercel env `MINIMAX_API_KEY`)

---
> Source: [JoyyyceD/xgrower-extension](https://github.com/JoyyyceD/xgrower-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
