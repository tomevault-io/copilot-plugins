## ai-linux-webapp-wrapper

> This repository is a small Electron wrapper template for Linux webapps.

# Agent Rules

## Purpose

This repository is a small Electron wrapper template for Linux webapps.
TradingView is the default example.

## Hard Rules

- Do not add secrets, cookies, tokens, local profile paths, or personal URLs.
- Do not enable `nodeIntegration` in the renderer.
- Do not disable `contextIsolation`.
- Do not disable `sandbox`.
- Do not allow arbitrary non-HTTPS external navigation.
- Prefer config changes in `app.config.json` over logic changes.
- Keep the wrapper small and auditable.

## Required Checks

Run before finishing code changes:

```bash
npm run verify
```

If dependencies are not installed yet, run:

```bash
npm install
npm run verify
```

## Review Focus

- Navigation must stay restricted to configured HTTPS hosts.
- External links must open in the default browser.
- Permission requests must be denied by default.
- User customization must not require editing security logic.

---
> Source: [trueugenee/ai-linux-webapp-wrapper](https://github.com/trueugenee/ai-linux-webapp-wrapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
