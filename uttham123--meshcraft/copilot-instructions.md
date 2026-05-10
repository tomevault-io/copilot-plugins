## meshcraft

> This repo is the **MeshCraft** skill system for building Figma screens in the MESH Design System (Meesho).

# MeshCraft — Claude Working Instructions

This repo is the **MeshCraft** skill system for building Figma screens in the MESH Design System (Meesho).

---

## Non-negotiable rules for every Figma task

### 1. Always invoke `/meshcraft` before starting any Figma screen work
Type `/meshcraft` at the start of every Figma-related task. The command embeds `meshInstance()`, the full component key index, and the layer map — do not start writing plugin code without it.

### 2. Never draw components from scratch
Every button, badge, appbar, tab, chip, snackbar, banner, switch, and icon **must** be imported from the MESH Library using `figma.importComponentByKeyAsync(key)`. Do not create rectangles, frames, or shapes to represent real DS components. No exceptions.

### 3. Always bind T2 semantic tokens after every import
After `importComponentByKeyAsync` + `createInstance()`, call `meshInstance()` to auto-bind T2 fills. Every layer that carries a background, text, or border fill must be bound to a variable in the **T2 — Semantic** collection. Never leave hardcoded fills on components you placed.

### 4. Use the correct component key — always check the index first
Before writing any import call, look up the exact variant key in:
- `.claude/commands/meshcraft.md` — inline index (always available via `/meshcraft`)
- `mesh-component-keys.md` — full master index with every variant

If a component isn't pre-mapped, follow the on-demand inspection workflow in `/meshcraft` — import the representative key, `dumpTree()` the instance, identify the fill layers, add to `MESH_LAYER_MAP`, then proceed.

### 5. Text is set in DM Sans; Mier B02 is applied via Font Fixer
The MCP plugin runs in Figma's cloud context where only Google Fonts are available. Set text content using `opts.text` in `meshInstance()`. After the script completes, the user runs the **MESH Font Fixer** local plugin (`mesh-font-fixer/`) once to convert all DM Sans → Mier B02.

### 6. All fills must reference T2 — Semantic variables
T2 tokens live in the **T2 — Semantic** collection in the Sandbox file (`u6xWbRPNRJgem7EwDrKfqr`). Use `_resolveT2(tokenPath)` from the `meshInstance()` utility. Never hardcode `{ r, g, b }` color values in fills for UI components.

---

## File map

| File | Purpose |
|------|---------|
| `.claude/commands/meshcraft.md` | Slash command — paste-ready `meshInstance()` + full key index |
| `mesh-component-keys.md` | Complete index of all MESH Library component variant keys |
| `mesh-layer-map.json` | Maps component variants → layer names → T2 token paths |
| `mesh-instance.js` | The `meshInstance()` utility (source of truth for the embedded version in `/meshcraft`) |
| `mesh-demand-index.md` | Fast lookup for components not yet pre-mapped — inspect on first use |
| `mesh-font-fixer/` | Local Figma dev plugin for DM Sans → Mier B02 conversion |
| `mesh-tokens/` | T1 + T2 variable definitions |

---

## Figma context

| Item | Value |
|------|-------|
| Sandbox file | `u6xWbRPNRJgem7EwDrKfqr` |
| MESH Library file | `i2MPgsD7m0EUVJ9MbaFyZH` |
| T2 collection name | `T2 — Semantic` |
| T1 collection name | `T1 — Primitives` |
| Mobile frame width | 390px |
| Font (design) | Mier B02 (Demi / Book / Bold) |
| Font (plugin context) | DM Sans (proxy; Font Fixer converts after) |

---

## Token model (3-tier)

```
T1 — Primitives   raw color/spacing/radius values (e.g. purple-600 = #9F2089)
       ↓ aliased
T2 — Semantic     role-based tokens (e.g. color/interactive/primary → T1/purple-600)
       ↓ bound via setBoundVariableForPaint
Components        instances from MESH Library with T2 fills applied
```

---

## Quick T2 token reference

| Token | Role |
|-------|------|
| `color/background/primary` | Screen / appbar / card bg |
| `color/background/secondary` | Chips, neutral fills |
| `color/background/tertiary` | Disabled / inactive |
| `color/background/inverse` | Dark fill |
| `color/text/primary` | Main body text |
| `color/text/secondary` | Captions, metadata |
| `color/text/inverse` | White text on dark/Jamun bg |
| `color/text/brand` | Jamun brand text (active) |
| `color/text/link` | Link actions |
| `color/interactive/primary` | Jamun (#9F2089) — CTAs, indicators |
| `color/border/default` | Dividers |
| `color/feedback/positive/surface` | Green bg |
| `color/feedback/positive/text` | Green text |
| `color/feedback/error/surface` | Red bg |
| `color/feedback/error/text` | Red text |
| `color/feedback/warning-soft/surface` | Yellow bg |
| `color/feedback/warning-strong/surface` | Orange bg |
| `color/ai/surface` | AI / Highlight bg |
| `color/ai/text` | AI text |

---
> Source: [Uttham123/meshcraft](https://github.com/Uttham123/meshcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
