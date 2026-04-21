## mc-site

> Static HTML/CSS/JS site built with Vite 6, deployed on Cloudflare Pages. Vanilla ES modules, no framework.

# mc_site — Merge Combinator Public Site

Static HTML/CSS/JS site built with Vite 6, deployed on Cloudflare Pages. Vanilla ES modules, no framework.

**Boundary rule:** If it requires auth, it lives on `guild.mergecombinator.com`. mc_site is public-only.

## Hosts

- `mergecombinator.com` — this repo (public marketing, /access)
- `guild.mergecombinator.com` — authenticated SPA (source: `signal-incubator/apps/console`)
- `api.mergecombinator.com` — CF Workers API
- `via.mergecombinator.com` — VIA identity provider (Authentik, OIDC)

## Dev Commands

```bash
npm run dev / npm run build / npm run preview
```

## Auth

VIA (Authentik) OIDC — mc_site does NOT store tokens; auth redirects to Guild. PKCE flow handled by `cloudflare/auth.js`.

## Design (C2UX)

Dark-native, Inter font, 2px radius. Tokens: `--blue` (#3b82f6), `--offwhite`, `--gray-light`, `--gray-medium`. Theme: `js/theme.js`, light mode: `styles/light-theme.css`.

## RTMX

`rtmx status` / `rtmx backlog` / `rtmx health` from project root.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/m4ndolore) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
