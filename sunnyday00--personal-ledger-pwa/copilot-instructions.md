## personal-ledger-pwa

> ﻿# Project Instructions

﻿# Project Instructions

## Documentation

- When changing project features, behavior, workflows, deployment steps, or architecture, update `README.md` in the same turn.
- Keep `README.md` aligned with the current implementation. Do not leave functional changes undocumented.

## Cloudflare Worker

- Files under `cloudflareworker/` are the Cloudflare Worker side of this project.
- If `cloudflareworker/` code or configuration is modified, deploy the Worker after the change in the same turn when possible.
- Use the Worker name `personal-ledger-sync`.
- Deploy with:
  - `npx wrangler deploy --config cloudflareworker/wrangler.toml`

## Verification

- After code changes, do not perform browser click or interaction testing yourself.
- If browser click or interaction testing is needed, provide the exact steps and expected result so the user can test manually.

## Git

- Git commit messages for this repository must be written in Chinese.

---
> Source: [SunnyDay00/Personal-Ledger-PWA](https://github.com/SunnyDay00/Personal-Ledger-PWA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
