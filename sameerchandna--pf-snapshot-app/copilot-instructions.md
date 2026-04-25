## pf-snapshot-app

> A local-only personal finance snapshot and projection tool. Users enter their income, expenses, assets, and liabilities to see a current financial snapshot, run multi-year projections, and model "what-if" scenarios. No cloud, no transaction tracking, no advice — purely observational.

# PF Snapshot App — Claude Guide

## What this app is
A local-only personal finance snapshot and projection tool. Users enter their income, expenses, assets, and liabilities to see a current financial snapshot, run multi-year projections, and model "what-if" scenarios. No cloud, no transaction tracking, no advice — purely observational.

## Read before making any significant change
- [docs/CODEBASE_MAP.md](docs/CODEBASE_MAP.md) — file paths, types, functions, navigation routes (read this first)
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — data flow and ownership model (Snapshot → Projection → Scenarios → Attribution)
- [docs/INVARIANTS.md](docs/INVARIANTS.md) — correctness rules that must never be violated
- [docs/CHANGE_PROTOCOL.md](docs/CHANGE_PROTOCOL.md) — when docs must be updated before implementation
- [docs/THEME.md](docs/THEME.md) — design token system
- [docs/SYSTEM.md](docs/SYSTEM.md) — product scope and boundaries

---

## UI Rules (mandatory, no exceptions)

### Spacing
- Screens use `layout.*` — `layout.screenPadding`, `layout.sectionGap`, `layout.screenPaddingTop` — from `ui/layout.ts`
- Components use `spacing.*` — `spacing.sm`, `spacing.md`, `spacing.xl`, etc. — from `ui/spacing.ts`
- No hardcoded numeric values: `padding: 16`, `marginBottom: 12`, `gap: 8` are all forbidden

### Section headers
- `SectionHeader` — financial sections with meaning (Snapshot, Cash Flow, Projection)
- `GroupHeader` — structural grouping (Assets, Accounts, Settings items)
- No custom title styles, no inline header typography

### Formatters
- All currency and percentage formatting from `ui/formatters.ts`
- Snapshot screens use full currency (`formatCurrencyFull`), Projection screens use compact (`formatCurrencyCompact`) — do not mix
- No local formatting helpers

### Typography
- All text styles use `theme.typography.*` spreads — never write `fontSize`, `fontWeight`, or `lineHeight` as bare numbers
- In components/screens with `makeStyles(theme)` or inline styles: `...theme.typography.label`, `...theme.typography.body`, etc.
- In static module-level `StyleSheet.create()`: import `{ typography }` from `'../ui/theme/theme'` and spread `...typography.body` etc.
- Available tokens: `header`, `sectionTitle`, `groupTitle`, `bodyLarge`, `body`, `bodySmall`, `caption`, `valueLarge`, `value`, `valueSmall`, `button`, `input`, `label`
- `fontWeight: '500'` does not exist in the token set — use `'400'` or `'600'`
- Exception: Victory chart `tickLabels`/`labels` props (SVG API, no spread support) — use `theme.typography.X.fontSize` directly

### Border radius
- Always `theme.radius.*` or the standalone `radius.*` import — never write `borderRadius: 8` etc.
- In `makeStyles(theme)` or inline: `theme.radius.medium`, `theme.radius.pill`, etc.
- In static StyleSheets: import `{ radius }` from `'../ui/theme/theme'`
- Scale: `none=0`, `small=4`, `base=6`, `medium=8`, `card=10`, `large=12`, `modal=14`, `pill=24`
- Exception: geometric half-radii (e.g. `borderRadius: height / 2` for circles) — leave as computed values

### Colors
- Always `theme.*` tokens from `useTheme()` — no hex values, no hardcoded colours
- No opacity-based pressed states — use background colour changes instead

---

## Architecture rules (things that must not be violated)

- **Snapshot** is the single source of truth. Only mutated via explicit `SnapshotContext` setters. Never touched by projection or scenario logic.
- **Projection** is ephemeral — computed from Snapshot + assumptions, never persisted, never mutates Snapshot.
- **Scenarios** modify projection inputs only — reversible, one active at a time, fall back to baseline if invalid.
- **Attribution** is observational — never mutates any state.
- **Flow vs Stock** — flows affect stocks only through explicit rules. Monthly surplus does not auto-accumulate into asset balances.
- **Doctrine changes** (anything altering system meaning or invariants) require updating the relevant docs/ file before writing code.

---

## New screen checklist
When building a new screen, follow this pattern:

1. Wrap in `DetailScreenShell` for standard screen layout
2. Use `ScreenHeader` for the top navigation bar
3. Use `SectionHeader` for financial sections, `GroupHeader` for structural grouping
4. Spacing from `layout.screenPadding`, `layout.sectionGap` (screens) or `spacing.*` (components)
5. All formatting via `formatters.ts`
6. If it's a collection screen, provide a `renderRow` override — no default row renderer exists
7. Use `CollectionRowWithActions` as the base row primitive for editable lists
8. Follow existing navigation patterns: stack navigator inside the relevant tab
9. All text styles via `...theme.typography.*` spreads — no bare `fontSize`/`fontWeight` numbers
10. All corner radii via `theme.radius.*` — no bare `borderRadius` numbers
11. For screens not using `DetailScreenShell`, wrap in `<SketchBackground color={palette.accent}>` (not `palette.bg`) — this gives the correct hatched look with the tab's accent colour

---

## Safe to change
- UI styling and layout of any screen
- Detail screens and their row configurations
- Formatters, spacing tokens, theme colours
- Navigation structure

## Do not touch without care
- `engines/projectionEngine.ts` — core simulation logic
- `engines/loanEngine.ts` — amortisation and loan calculations
- `engines/computeA3Attribution.ts` — A3 attribution reconciliation math
- `context/SnapshotContext.tsx` setters — mutation boundary
- Any core invariant listed in `docs/INVARIANTS.md`

If a proposed change touches these, re-read `docs/INVARIANTS.md` and `docs/CHANGE_PROTOCOL.md` first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sameerchandna) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
