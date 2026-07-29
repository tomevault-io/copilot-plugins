## parchi

> - Capture user interaction context (events + screenshots) for sidepanel recording workflows.

# Recording Module Notes

## Purpose
- Capture user interaction context (events + screenshots) for sidepanel recording workflows.

## Boundaries
- Keep Chrome/runtime API calls inside this module; do not couple recording logic to sidepanel DOM state.
- Shared recording types live in `/Users/sero/projects/browser-ai/packages/shared/src/recording.ts`.
- Message contracts used by background/sidepanel should be typed and imported from shared paths.

## Rules
- New files should stay under 300 lines where practical; split helpers when the coordinator grows.
- Preserve restricted URL safeguards and cleanup behavior for timers/listeners.
- Run `npm run typecheck` and `npm run build` after changes.

---
> Source: [0xSero/parchi](https://github.com/0xSero/parchi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
