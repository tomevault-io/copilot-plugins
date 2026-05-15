## mailpit-app

> Standalone Alpine service that downloads and runs [axllent/mailpit](https://github.com/axllent/mailpit) — SMTP sink + web UI. No user source to iterate on.

# mailpit-app

Standalone Alpine service that downloads and runs [axllent/mailpit](https://github.com/axllent/mailpit) — SMTP sink + web UI. No user source to iterate on.

## Zerops service facts

- HTTP ports: `8025` (web UI, `httpSupport: true`); `1025` (SMTP, plain TCP)
- Siblings: —
- Runtime base: `alpine@latest`

## Zerops

No dev iteration loop — the app is an upstream binary. Changes in this repo only affect `download-mailpit.sh`. Each change requires a full build+deploy through the **Zerops development workflow via `zcp` MCP tools**.

## Notes

- Import snippet lives in `README.md`; no standalone project import YAML in this repo.

---
> Source: [zerops-recipe-apps/mailpit-app](https://github.com/zerops-recipe-apps/mailpit-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
