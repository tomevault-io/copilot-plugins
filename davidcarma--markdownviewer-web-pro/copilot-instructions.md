## markdownviewer-web-pro

> - **No external runtime dependencies**: do not load JS/CSS/fonts from CDNs (no `http(s)://` in `<script src>`, `<link href>`, or CSS `@import`).

## Project rules (hard requirements)

- **No external runtime dependencies**: do not load JS/CSS/fonts from CDNs (no `http(s)://` in `<script src>`, `<link href>`, or CSS `@import`).
- **Vendored libraries only**: any third-party runtime library must be committed into this repo and referenced via relative paths.
- **Offline-first**: core editor must run locally with **zero network access**. No background fetches for app functionality.

## Cloud integrations (permitted external calls)

Optional cloud integrations (Google Drive, GitHub, etc.) may make external API calls **provided**:

1. **Serverless only** - all API calls happen directly from the browser. No backend server, proxy, or cloud function required.
2. **Graceful degradation** - the app works fully offline; integration UI is hidden or disabled when unavailable.
3. **User-initiated** - network calls only happen after an explicit user action (sign in, open, save). No silent background fetches.
4. **Auth libraries loaded async/defer** - external auth scripts (e.g., Google GIS) must not block page load.

### Production analytics exception

Google Analytics may be used on the production site **only** if all of the following are true:

1. **Production-only** - analytics must not load on localhost, file URLs, or non-production preview hosts.
2. **Explicit consent first** - analytics must stay off until the user explicitly accepts analytics tracking.
3. **No feature dependency** - the editor and file system must continue working fully without analytics.
4. **Async only** - analytics scripts must load asynchronously and must not block page render.
5. **Privacy disclosure** - the site's privacy page must disclose analytics usage, consent behavior, and the provider.

### Currently permitted external endpoints

| Endpoint | Purpose |
|----------|---------|
| `https://accounts.google.com/gsi/client` | Google Identity Services OAuth (Drive integration) |
| `https://www.googletagmanager.com/gtag/js` | Google Analytics tag loader for the production site |
| `https://www.google-analytics.com/*` | Google Analytics event collection |
| `https://region1.google-analytics.com/*` | Regional Google Analytics event collection |
| `https://github.com/login/device/code` | GitHub device-flow auth |
| `https://github.com/login/oauth/access_token` | GitHub device-flow token exchange |
| `https://api.github.com/*` | GitHub REST API (repos, gists, contents) |
| `https://oauth2.googleapis.com/token` | Google OAuth token exchange and refresh (Drive auth code flow) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidcarma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
