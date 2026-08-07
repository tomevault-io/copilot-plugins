## glasskit-ui

> This is the public OSS monorepo for the GlassKit UI ecosystem. **Read

# glasskit-ui — agent guide

This is the public OSS monorepo for the GlassKit UI ecosystem. **Read
[`START-HERE.md`](./START-HERE.md) first**, then [`PLAN.md`](./PLAN.md) for the
full strategy and 7-phase plan.

## Before you build

- **`apps/web` is a modified Next.js 16.** APIs, conventions, and file structure
  may differ from your training data. Read the relevant guide in
  `node_modules/next/dist/docs/` before writing any Next code, and heed
  deprecation notices.
- **Platform truth = [`docs/platform-audit-2026-06.md`](./docs/platform-audit-2026-06.md)** —
  what the Display's Web Apps runtime actually delivers (verified against
  Meta's docs/toolkit, 2026-06): arrows/Enter keydowns, back as history-pop →
  `popstate` (OS v125.1+; Escape is desktop-sim only), no custom gesture
  events, required 600×600 + `mrbd-web-app-capable` meta tags.
- The SDK to extract lives in a sibling repo:
  `/Users/jarrius/Documents/GitHub/glasskit-boilerplate/packages/glasses-ui`.
  Preserve its public API verbatim (see `START-HERE.md` §2).
- **Component aesthetic = [`docs/design/meta-hud-language.md`](./docs/design/meta-hud-language.md)** —
  the current, owner-approved visual language (2026-06): a calm dark base, **popping
  surfaces**, and **tasteful gradient icon plates**. Build styled components to that
  spec. [`docs/design/apple-feel.md`](./docs/design/apple-feel.md) is the earlier
  additive-lens reference (type ramp, motion, RTL still apply); where the two
  conflict on fills/gradients/`#000`, the Meta-HUD spec wins.

## Hard rules

- **No `Co-Authored-By: Claude`** (or any agent attribution) trailer on commits.
- **No inline CSS** (`style={{}}`). Both site and lens are Tailwind utilities +
  `--gk-*` design tokens (the shadcn model). `styles.css` holds only primitives
  (tokens, `.focusable`, `@keyframes`, pseudo-element + world-anchored rules,
  the `surface`/`btn-primary`/`press-scale` `@utility` recipes); a component's
  layout/color lives in its `.tsx` as utilities. Lens = logical RTL utilities
  only. See CLAUDE.md "Styling" + `/docs/theming`.
- **Do not create the public GitHub repo or publish to npm** without the owner's
  explicit go-ahead. CI + Changesets get configured but stay inert until enabled.
- **Premium-surfaces direction (supersedes the old "additive lens is sacred"
  rule).** The owner chose a premium app-UI look over literal see-through-device
  truth (2026-06): a calm dark base (`#0b0e16`→`#090b11`, faint cool top
  highlight), surfaces that *pop* (top-lit gradient fill + 1px light edge + soft
  depth), and gradient icon plates. Pure `#000`-transparency, "no fills", and "no
  shadows/gradients on the 600×600 surface" no longer apply — that was the
  additive model, now history. Still avoid `backdrop-filter`/blur on the lens
  content itself (use surface fills, not frosted glass). See
  `docs/design/meta-hud-language.md`.
- **RTL from day one:** logical CSS only; never mirror world-anchored components
  (DirectionArrow, Compass, Pin), see PLAN.md "Internationalization."
- **Propagate every component change across ALL AI surfaces.** The skill, the
  MCP, and `llms.txt` are the AI's source of truth and must never drift from the
  code. Whenever you add, rename, remove, or change a component's props/variants,
  update every surface in the *same* change:
  1. the component JSDoc in `registry/ui/*.tsx` (feeds `registry.json` + `llms.txt`),
  2. the docs — `apps/web/lib/component-docs.tsx` (props table, usage, summary) +
     `apps/web/content/docs/**`,
  3. the agent skill + rules in `packages/cli/src/agents.ts` (`AGENTS_MD`,
     `SKILL_MD`, the Cursor + Copilot bodies),
  4. the MCP guidelines in `packages/mcp/src/index.ts` (`GUIDELINES`),
  5. the `llms.txt` source `apps/web/llms-template.md`,
  6. the design spec `docs/design/meta-hud-language.md` (only if the look changes).
  Then regenerate: `pnpm build:registry` (rebuilds `registry.json`, `/r/*.json`,
  `llms.txt`) and `pnpm --filter @glasskit-ui/cli build` (rebuilds the shipped
  skill in `dist/`). Finally grep the old/removed name across `packages`,
  `apps/web/content`, and `docs` to confirm zero stragglers.

## Conventions

pnpm (`pnpm@10.8.0`) + turborepo. Mirror the boilerplate's `tsconfig.base.json`,
`turbo.json`, `prettier.config.mjs`, and CI shape (documented in
`START-HERE.md` §3). Run graphify (`graphify update .`) after code changes.

---
> Source: [GlassKitApp/glasskit-ui](https://github.com/GlassKitApp/glasskit-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
