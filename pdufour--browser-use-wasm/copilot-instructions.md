## green-circle-commit

> Green Circle commit message format for perf wins


# Green Circle commits

Only commit E2E perf wins when `npm run test` passes 3/3.

**Message format:** `{taskMs}ms 🟢 {E2EModelId} {description}`

- **taskMs** — best (lowest) Run task inference ms across the 3× run, from local `e2e-results.txt` (gitignored) / `[perf:e2e] task inference` (Green Circle metric). Not task wall, not load time.
- **E2EModelId** — model under test in `npm run test` (today **ShowUI-2B** only; see `blackbox-e2e.mdc`).
- **description** — short what changed.

Examples:

- `2100ms 🟢 ShowUI-2B navigation-mode Run task`

Do not omit the model id. Do not use wall ms for the prefix number.

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
