## systemawakening

> node build.js        # compiles engine.ts → dist/engine.js

# Claude Code — System Awakening

## CRITICAL: Rebuild the bundle after every source change

```bash
node build.js        # compiles engine.ts → dist/engine.js
```

Commit **both** source files and `dist/engine.js`. TypeScript changes have zero effect until the bundle is rebuilt.

---

## Project structure

| Path | Purpose |
|------|---------|
| `engine.ts` | Boot — wires modules, starts game |
| `src/core/` | Interpreter, parser, evaluator, state, DOM helpers |
| `src/systems/` | Saves, undo, inventory, skills, items, journal, glossary, procedures |
| `src/ui/` | Narrative, status panel, overlays |
| `dist/engine.js` | **Deployed bundle** — commit pre-built |
| `startup.txt` | Global vars, stats, scene list |
| `skills.txt / items.txt / procedures.txt / glossary.txt` | Game data registries |

---

## Commands

```bash
node build.js          # bundle
npx tsc --noEmit       # type-check
npm run lint           # scene linter
npm test               # unit tests
npm run test:e2e       # Playwright
npm run dev            # dev server :3000
```

---

## Architectural rules

- **No circular imports.** UI → core only. Inject callbacks at boot to break cycles.
- **XSS:** all strings through `escapeHtml()` before `innerHTML`. No `eval()`.
- **Saves:** SA1 format, version 9. Changing `SAVE_VERSION` in `saves.ts` invalidates old saves.
- **Expressions:** parse errors return `0` (fail-closed). Never silently pass broken conditions.
- **Char creation:** never call `focus()` on a name input inside the rAF that sets defaults — Chrome fires focus async and wipes the value. Focus the pronoun card instead.

---

## Deployment

GitHub Pages from `main` root. No CI build — `dist/engine.js` must be committed. Hard-refresh (Ctrl+Shift+R) if live site looks stale.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShaneEnochs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
