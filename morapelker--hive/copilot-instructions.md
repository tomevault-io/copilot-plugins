## hive

> Read this before editing anything. The token foundation has ALREADY landed in

# Orca migration — shared implementation conventions

Read this before editing anything. The token foundation has ALREADY landed in
`src/renderer/src/styles/globals.css` and `src/renderer/src/lib/themes.ts`:

- `--primary` is now NEUTRAL: `#e5e5e5` in dark (fg `#171717`), `#171717` in
  light (fg `#fafafa`). It is no longer purple. Any `bg-primary/N` tint or
  `text-primary` accent that relied on hue is broken until you fix it.
- `--ring` is `#737373` (dark) / `#a1a1a1` (light). `--accent` is `#404040`
  dark (a strong hover fill), `--secondary`/`--muted` `#262626`.
- `--border` is `rgb(255 255 255 / 0.07)` in dark — hairline. Do not dilute it
  further (`border-border/50` on dark = invisible).
- New tokens exist and are aliased for Tailwind:
  `bg-worktree-sidebar`, `text-worktree-sidebar-foreground`,
  `bg-worktree-sidebar-accent`, `border-worktree-sidebar-border`,
  `ring-worktree-sidebar-ring` (dark: #2a2a2a / #fafafa / #353535 / 7% white).
  The LEFT sidebar and the titlebar's left segment use these; the RIGHT sidebar
  keeps `bg-sidebar` (#171717).
- Geist is the UI font (already wired as `--font-sans`); body has
  `letter-spacing: 0.01em`. JetBrains Mono remains the terminal font.

## Canonical recipes (copy verbatim)

- **Focus ring** (everything focusable):
  `outline-none focus-visible:ring-[3px] focus-visible:ring-ring/50 focus-visible:border-ring`
  Never `ring-2`, never `ring-offset-*`, never `/24`.
- **Glass menu/popover** (dropdown, context menu, popover, hover-card, toasts,
  command palette):
  `rounded-[11px] border border-black/14 dark:border-white/14 bg-[rgba(255,255,255,0.82)] dark:bg-[rgba(0,0,0,0.72)] backdrop-blur-2xl shadow-[0_16px_36px_rgba(0,0,0,0.24),inset_0_1px_0_rgba(255,255,255,0.04)]`
- **Glass dialog** (Dialog/AlertDialog/HelpOverlay):
  content `rounded-[11px] border border-black/14 dark:border-white/14 bg-background/96 backdrop-blur-xl shadow-[0_24px_64px_rgba(0,0,0,0.35)]`;
  overlay `bg-black/50 backdrop-blur-[2px]`.
- **Menu row**: `rounded-[6px] px-2 py-1.5 text-[13px] gap-2` with
  `focus:bg-black/6 dark:focus:bg-white/8` (soft wash, not solid `bg-accent`).
- **Menu label**: `text-[11px] font-medium uppercase tracking-[0.06em] text-muted-foreground`.
- **Separator on glass**: `-mx-1 my-1 h-px bg-border`.
- **Tooltip**: INVERTED — `bg-foreground text-background rounded-md px-2 py-1 text-[11px] font-medium` with matching `fill-foreground` arrow, no border, no shadow.
- **Section header** (sidebar/panel): `text-[11px] font-semibold uppercase tracking-wider text-muted-foreground`.
- **Active tab**: bg `color-mix(in srgb, var(--foreground) 6%, var(--card))`
  (Tailwind: `bg-[color-mix(in_srgb,var(--foreground)_6%,var(--card))]`), plus a
  2px bottom bar `bg-[color-mix(in_srgb,var(--foreground)_60%,var(--card))]`.
- **Titlebar buttons**: icon `size-6 rounded-md text-muted-foreground hover:bg-accent hover:text-foreground`;
  text `h-6 px-2 text-[12px] font-medium rounded-md`.
- **Chrome density**: chrome/UI text 13px (`text-[13px]`), meta 11px, micro 10px.
  Chat/message body text stays `text-sm`.
- **Shadows**: none in chrome. Only the glass recipes above carry shadows.
  Delete `shadow-xs/5`, `shadow-sm`, `shadow-md`, `shadow-lg`, `shadow-2xl`
  and every `before:*` inner-sheen pseudo-element you find in chrome.

## Status vocabulary (semantic colors — KEEP/USE these)

- working: `yellow-500` (spinner ring — a rotating `border-2 border-yellow-500
  border-t-transparent rounded-full` div with `animation: spin 1s steps(12,end) infinite`,
  or keep existing Loader2 but color it `text-yellow-500`)
- done/ready/merge: `emerald-500` (merged states that were purple → emerald)
- question/permission/attention: `orange-500` (amber-500 acceptable where already used)
- plan mode: violet-500 ramp (pills, progress, plan bubbles) — KEEP violet ONLY
  for plan-mode semantics; purple-500 usages for plan → normalize to violet-500
- build mode: blue-500; unread indicators: blue-500 dots
- remote/teleport: sky-500; monitors: cyan-500; errors/destructive: red
- vendor/file-type identity colors (Figma purple, language icons, file icons): KEEP

## primary-flip decision rules

- Inverted selected chips / solid CTAs / active-tab underlines / search
  highlights → KEEP neutral primary/foreground (that's the orca look), but
  verify contrast and prefer `bg-foreground text-background` phrasing where the
  inventory says so.
- Unread dots, "Working" text/spinners, update/attention pills, drop-zone and
  drag indicators → move to the status vocabulary above (NOT primary).
- Every `bg-primary/5`–`/25` wash → replace with `bg-secondary`, `bg-accent/50`
  or a status tint per your inventory entry. Never leave a low-opacity primary
  wash in place.

## Process rules

- Edit ONLY files in your assigned ownership list. If an inventory entry names
  a file outside your list, SKIP it and note it in your final report.
- `src/renderer/src/components/ui/*` primitives are being fixed in a prior wave —
  assume Button/Input/Dialog/Popover/Tooltip/Menus already carry the orca
  recipes; do not re-apply glass on top of them at call sites (remove per-call
  overrides that now conflict, e.g. duplicate shadows/radii).
- Update co-located tests that assert on class strings you change (e.g.
  UsageIndicator.test.tsx `border-purple-500`, TicketModelBadge.test.tsx
  `border-violet-500`). Do not run test suites yourself.
- Do not run build/typecheck/tests — the orchestrator verifies centrally.
- Do not commit.
- Keep code style local: className strings inline like the surrounding code,
  `cn()` merging preserved, no new dependencies.
- Reference mockups: design/redesign/01-orca.html (canonical), 06-board.html,
  07-terminal-usage.html, 08-pr.html, 09-connections.html.

---
> Source: [morapelker/hive](https://github.com/morapelker/hive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
