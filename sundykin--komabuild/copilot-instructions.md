## komabuild

> - Do not open the frontend in a normal browser for UI smoke tests or visual verification.

# Project Agent Rules

## Electron Visual Verification

- Do not open the frontend in a normal browser for UI smoke tests or visual verification.
- This app must be inspected through Electron's custom Chromium remote debugging port.
- The port is configured in `electron/main.ts`:
  - env override: `KOMA_ELECTRON_REMOTE_DEBUGGING_PORT`
  - default: `9333`
- Use DevTools Protocol / chrome-devtools-mcp against:
  - `http://127.0.0.1:${KOMA_ELECTRON_REMOTE_DEBUGGING_PORT:-9333}`
- If the port is not listening, start the Electron dev app first; do not fall back to opening `localhost` / Vite in a regular browser unless the user explicitly asks for that fallback.

---
> Source: [Sundykin/KomaBuild](https://github.com/Sundykin/KomaBuild) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
