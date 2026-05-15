## mccullough-digital

> This repository is the public McCullough Digital WordPress block-theme and client-delivery surface. Keep it recruiter/client friendly: source, docs, tests, case-study proof, and build instructions should be easy to inspect without local workstation context.

# Agent Notes

This repository is the public McCullough Digital WordPress block-theme and client-delivery surface. Keep it recruiter/client friendly: source, docs, tests, case-study proof, and build instructions should be easy to inspect without local workstation context.

## Build And QA

1. Run `npm install`.
2. Use `npm run start` while developing blocks.
3. Run `npm run build` before committing production bundles.
4. Run the Playwright smoke suite with `npm run test:e2e -- --project=chromium` when a local WordPress site is available.

## Documentation

- `README.md` is the public portfolio overview.
- `readme.txt` is the WordPress theme changelog.
- `docs/qa-release-log.md` is the rolling QA/release verification log.
- `docs/onmark-case-study.md` is the public OnMark proof artifact.
- `docs/launch-readiness-checklist.md` and `docs/ga4-launch-checklist.md` document reusable delivery checklists.

## Repository Hygiene

- Keep generated package archives, one-off patch files, local scratch pads, and standalone preview exports out of the public tree.
- Do not commit client-private source material, raw credentials, production database exports, or local machine paths.
- Build assets under `build/blocks/` are intentionally tracked because this is a WordPress theme surface.

---
> Source: [GalToast/mccullough-digital](https://github.com/GalToast/mccullough-digital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
