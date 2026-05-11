## 17jewelswebsite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static marketing and auth website for the **17 Jewels** iOS pocket watch collection app, deployed on Vercel at `https://17jewels.app`.

No build step, no framework, no package manager — just plain HTML/CSS/JS files served directly.

## Deployment

```bash
# Deploy via Vercel CLI
npm i -g vercel
vercel
```

Or push to the GitHub repo and Vercel auto-deploys via the GitHub integration.

## Pages and routing

Routes are defined in `vercel.json` — clean URLs map to `.html` files:

| URL | File |
|-----|------|
| `/` | `index.html` |
| `/confirmed` | `confirmed.html` |
| `/reset-password` | `reset-password.html` |
| `/privacy` | `privacy.html` |
| `/terms` | `terms.html` |

## Architecture notes

- **CSS variables** — all colors/typography are defined as CSS custom properties in a `:root` block at the top of each page's `<style>` tag. The palette is: `--brown` (#2C1810), `--gold` (#D4AF37), `--cream` (#FAFAF8).
- **No shared CSS** — each page is fully self-contained (styles inlined in `<head>`). When changing styles, update each page independently.
- **Supabase integration** — `reset-password.html` loads `@supabase/supabase-js` from CDN and uses it to handle password reset tokens. The Supabase URL and anon key are hardcoded in the script block. The confirmed page (`confirmed.html`) is a static success screen with no JS logic.
- **Images** — served from `/images/`. The `og-image.png` and `apple-touch-icon.png` are referenced in HTML but not yet in the repo.

## Supabase configuration (external)

After any domain change, update in Supabase Dashboard → Authentication → URL Configuration:
- Site URL: `https://17jewels.app`
- Redirect URLs: `https://17jewels.app/reset-password`, `pocketwatchapp://**`

---
> Source: [dorosp81/17JewelsWebsite](https://github.com/dorosp81/17JewelsWebsite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
