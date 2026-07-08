## sabihr

> This file documents the visual language we've converged on for the SabiHR app. Apply these patterns whenever adding or modifying UI. Auto-loaded by Claude Code every session.

# SabiHR — Design System Rules

This file documents the visual language we've converged on for the SabiHR app. Apply these patterns whenever adding or modifying UI. Auto-loaded by Claude Code every session.

## Background

- **All page-level backgrounds**: `bg-[#f7fbff]` (Tailwind `blue-50` shade — soft light blue).
- Applied in: `src/components/layout/app-layout.tsx`, `src/components/layout/employee-layout.tsx`, and any standalone page that sets its own `min-h-screen bg-*`.
- Internal element tints (table hover, nested cards, chip backgrounds): use `bg-slate-50` or `bg-slate-100`, not a second blue shade — reserve blue for brand/primary use.

## Cards

Standard card shell for any content block (tables, KPIs, lists, forms):

```
rounded-2xl border border-slate-200/70 bg-white shadow-[0_1px_2px_rgba(15,23,42,0.04),0_1px_3px_rgba(15,23,42,0.05)]
```

- **Radius**: always `rounded-2xl` (not `rounded-xl` or `rounded-lg`) for top-level cards.
- **Border**: `border border-slate-200/70` (soft, low-contrast — the shadow does the lifting).
- **Shadow**: the long `shadow-[...]` literal above — promote to a utility if we add a Tailwind plugin.
- **Internal dividers** (between sub-sections or table rows): `border-slate-100` (lighter than the outer border).

Never use `border-[#efefef]` for new work — legacy pages still have it; replace on touch.

## Tables

Consistent with the **Employees** page table, which is the reference:

- **Outer**: card shell (see above) with `overflow-hidden`.
- **Toolbar row** (search + count + sort, above the table): `flex items-center gap-3 justify-between px-5 py-4 border-b border-slate-200/70`.
- **Thead row**: just `border-b border-slate-200/70`. No `bg-[#f8fafc]` / no gray header band.
- **Th cells**: `text-left font-medium text-[11px] uppercase tracking-wider text-slate-500 py-3 pr-5`. First column gets `pl-5`.
- **Tbody rows**: `border-b border-slate-100 last:border-0 hover:bg-slate-50/60 transition-colors`. Add `cursor-pointer` when rows navigate.
- **Td cells**: `py-4 pr-5`, first column `pl-5`. Use `py-4` for breathing room.
- **Checkbox column**: `w-12 pl-5 py-3`. Checkbox: `w-4 h-4 rounded border-slate-300`.

### Inside-cell patterns

- **Avatar + primary + secondary** (name + email / title / subtitle):
  ```
  <div className="flex items-center gap-3">
    <Avatar />
    <div>
      <p className="font-semibold text-slate-900 leading-tight">{primary}</p>
      <p className="text-xs text-slate-500 leading-tight mt-0.5">{secondary}</p>
    </div>
  </div>
  ```
- **Tag pill** (department, category, location): `inline-flex items-center px-2.5 py-1 rounded-md bg-slate-100 text-slate-700 text-xs font-medium`.
- **Status pill** (dot + label): `inline-flex items-center gap-1.5 px-2.5 py-1 rounded-md text-xs font-medium` with `<span className="w-1.5 h-1.5 rounded-full bg-{color}-500" />` inside. Status bgs:
  - active / success → `bg-emerald-50 text-emerald-700`
  - pending / warning → `bg-amber-50 text-amber-700`
  - onboarding / info → `bg-blue-50 text-blue-700` or `bg-violet-50 text-violet-700`
  - danger → `bg-rose-50 text-rose-700`
  - inactive / terminated → `bg-slate-100 text-slate-600`
- **Row overflow button**: `w-8 h-8 rounded-md text-slate-400 hover:text-slate-700 hover:bg-slate-100 inline-flex items-center justify-center` wrapping `<MoreHorizontal className="w-4 h-4" />`.

## Avatars

Always **solid fill + white text** (never pastel-with-dark-text):

```
w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold
```

- Sizing: `w-10 h-10` in tables; `w-9 h-9` or `w-8 h-8` for compact lists / nested rows.
- **Color per avatar** — rotate a 10-color palette hashed by the row's stable `id` so the color stays consistent across renders:
  ```
  const AVATAR_PALETTE = [
    "bg-blue-500", "bg-violet-500", "bg-teal-500", "bg-amber-500", "bg-rose-500",
    "bg-emerald-500", "bg-indigo-500", "bg-pink-500", "bg-orange-500", "bg-cyan-500",
  ];
  ```
- **System / non-person avatars**: `bg-slate-400` with white initials (e.g. "SY").
- **Profile photos**: render `<img className="w-10 h-10 rounded-full object-cover">` with `onError` → fall back to the initials avatar on load failure.

## KPI / Stat Cards

Row of 4 equal cards above a table or below a banner. Each card:

```
rounded-2xl border border-slate-200/70 bg-white px-5 pt-5 pb-5
shadow-[0_1px_2px_rgba(15,23,42,0.04),0_1px_3px_rgba(15,23,42,0.05)]
flex flex-col gap-7
```

Content:
1. Top row: icon well (left) + trend pill (right).
2. Bottom: big number (`text-3xl font-bold tracking-tight text-slate-900 leading-none`) + label (`text-sm text-slate-500 mt-2`).

- **Icon well** (uniform across all KPIs on a page): `w-10 h-10 rounded-xl bg-blue-50 flex items-center justify-center` with icon `text-blue-600 w-[18px] h-[18px]`. **Do not use per-card varied colors** (no rainbow of emerald/violet/amber/rose for KPIs on the same page) — keep the well color uniform, let the numbers do the differentiating.
- **Trend pill**: `inline-flex items-center gap-0.5 text-xs font-semibold`, green `text-emerald-600` for up with `<TrendingUp />`, red `text-rose-600` for down with `<TrendingDown />`. Format: `+N` or `-N` (add `%` suffix for rate metrics).

## Chip Filters (tabs replacement)

Horizontal pills for filtering a table view (e.g., department tabs):

```
h-9 px-4 rounded-full text-sm font-medium inline-flex items-center gap-1.5
active:   bg-blue-600 text-white
inactive: bg-white border border-slate-200 text-slate-700 hover:bg-slate-50
```

Optional count: `<span className="text-xs text-{white/80 | slate-400}">{count}</span>` inside.

## Primary / Secondary Buttons

- **Primary**: `h-10 rounded-lg bg-blue-600 hover:bg-blue-700 text-white font-semibold px-4`. Use for the single dominant CTA per row (e.g. `+ Add employee`).
- **Secondary / outline**: `h-10 rounded-lg border-slate-200 text-slate-700 font-semibold bg-white`. Use `variant="outline"` on our Button component. For Filters / Export / Cancel.
- **Link-style action in a card row**: `text-[11px] font-medium text-blue-600 hover:text-blue-700` (e.g., "View all", "Review").

## Banner / Hero

Top-of-page banner pattern (see `src/components/dashboard/hero-greeting.tsx` and Employees page):

```
rounded-2xl bg-gradient-to-br from-[#2563eb] via-[#3b82f6] to-[#1d4ed8] text-white
```

Structure:
1. Small backdrop-blur chip top-left: `rounded-full bg-white/15 backdrop-blur px-2.5 py-1 text-[11px] font-medium` with an icon and a breadcrumb like "People · Northwind Studio".
2. Title: `text-2xl sm:text-3xl font-bold tracking-tight`.
3. Subtitle: `text-sm text-white/85 max-w-xl leading-relaxed` describing what the user can do on this page.
4. Optional CTAs: white-filled primary + ghost secondary (both inside the banner).
5. Decorative floating rotated squares + blur glows on the right for depth.

## Typography

- Page title (in content, no banner): `text-3xl font-bold tracking-tight text-slate-900`.
- Card/section title: `text-base font-bold text-slate-900 tracking-tight` or `text-sm font-semibold text-slate-900` for compact.
- Card subtitle/description: `text-xs text-slate-500 mt-0.5`.
- Body text: `text-sm text-slate-700` for content, `text-slate-500` for meta/labels.
- Stat numbers: `text-3xl font-bold tracking-tight text-slate-900 leading-none`.

## Spacing

- Page container: `max-w-[1500px] space-y-5` (or `space-y-6` if lots of sections).
- Between sections: `space-y-5` or `space-y-6`.
- Inside cards: `px-5 py-4` for toolbar rows, `px-5 pt-5 pb-5` for KPI cards, `py-4 pl-5 pr-5` for table cells.
- Grid gap between sibling cards: `gap-4`.

## Iconography

- **Library**: `lucide-react` (default). Don't introduce a second icon library without renaming the wrapper component.
- **Default icon sizing**: `w-4 h-4` inline, `w-5 h-5` in buttons with text, `w-[18px] h-[18px]` in icon wells.
- **Icon color in neutral wells** (`bg-slate-50`): `text-slate-700`. In branded wells (`bg-blue-50`): `text-blue-600`.

## Page Layout Checklist

When building a new page under AppLayout:

1. Wrapper: `<div className="max-w-[1500px] space-y-5">`.
2. (Optional) Banner — see Banner section.
3. Top row with action buttons right-aligned (Filters, Export, + Add X).
4. KPI row — 4 stat cards (all with uniform blue-50 wells).
5. Table card — toolbar row + table.
6. Modals use: `rounded-2xl border border-slate-200/70 bg-white shadow-xl` + the inner padding pattern from Reassign modal in departments.

## Things to avoid

- `bg-[#f8fafc]` as a **page** background (use `bg-[#f7fbff]`). Keep it only for inner element tints.
- `border-[#efefef]` — replaced with `border-slate-200/70` (outer) and `border-slate-100` (inner).
- Pastel-100 + colored-text avatars (`bg-blue-100 text-blue-700`). Use solid `bg-{color}-500` + white text instead.
- Per-card rainbow of icon well colors on one page. Uniform well color, let numbers speak.
- `rounded-xl` for top-level cards (use `rounded-2xl`).
- Heavy gray table header bands (`thead tr bg-[#f8fafc]`). Headers sit flush on white with just `border-b border-slate-200/70`.
- `text-white` on gradient hero without providing a non-`text-` visual fallback for small screens — check banner still reads when the decorative shapes are hidden.
- Nesting a card inside another card without visually quieter internal styling. If you must, use `bg-slate-50 rounded-xl` on the inner to avoid a double-shadow look.

## Responsiveness

Target the layout to work at three sizes: **mobile 390×844 (iPhone 14/15)**, **tablet 768–1024**, **desktop ≥1024**. Use stock Tailwind breakpoints — `sm:640`, `md:768`, `lg:1024`, `xl:1280`. The sidebar is mobile-drawer below `lg`, fixed-rail at and above.

### Page-level patterns

- **Page wrapper**: always `max-w-[1500px] space-y-5` (or `space-y-6`). `AppLayout` handles horizontal padding (`p-4 lg:p-6`), so individual pages do NOT add their own.
- **Page header row** (title + action buttons): `flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4`. Action buttons sit below the title on mobile, beside it from `sm:` up. Keep the action button(s) full-width on mobile (`w-full sm:w-auto`) when there is only one CTA.
- **Page H1**: `text-2xl sm:text-3xl font-bold tracking-tight text-slate-900`. Never use bare `text-3xl` for an `<h1>`.
- **Section / card titles**: keep as-is (`text-base font-bold` / `text-sm font-semibold`) — they already read well at small sizes.

### KPI rows

- 4-up KPI cards: `grid grid-cols-2 lg:grid-cols-4 gap-4`. **2 across on mobile, 4 across from `lg:`** — never single-column (KPIs are short numbers and look lonely at full width).
- 3-up KPI cards: `grid grid-cols-1 sm:grid-cols-3 gap-4` (or `sm:grid-cols-2 lg:grid-cols-3` if you have detailed sub-content per card).
- KPI value (`text-3xl`): leave as-is — fits at 2-col mobile.

### Tables

- **Always** wrap `<table>` in `<div className="overflow-x-auto">`. Add `min-w-[Xpx]` on the table itself sized to the natural total column width (typically 720–960px) so columns don't squish before the user scrolls.
- Toolbar row above the table: `flex flex-col sm:flex-row sm:items-center gap-3 sm:justify-between`. Search input goes full-width on mobile (`flex-1` already does it inside the col).
- **Card-stack alternative** (only for tables that are the *primary* artifact of a page — Employees, Dashboard Activity, Payroll Runs): hide the table with `hidden md:block` on the scroll wrapper, and render a `md:hidden space-y-3 p-4` list of `rounded-xl border border-slate-200/70 bg-white p-4` cards next to it. Each card shows the primary cell (avatar + name + secondary), the status pill, and a row of `text-xs text-slate-500` meta lines. The remaining cells (department/location/joined) become inline `<dt>/<dd>` rows inside the card.
- Inline overflow buttons stay; row-click navigation still works via the card's `onClick`.

### Hero / banner

- Stack on mobile: outer flex is already `flex-col sm:flex-row`. Title `text-2xl sm:text-3xl`. Decorative right-hand shapes use `hidden sm:flex`.
- CTA row inside the banner: `flex flex-wrap gap-2` so two buttons wrap rather than overflow. Optionally make the primary `w-full sm:w-auto` if only one CTA.

### Modals / overlays

- Container: `fixed inset-0 z-50 flex items-end sm:items-center justify-center p-0 sm:p-4 bg-black/40 backdrop-blur-[2px]`.
- Modal card: `w-full sm:max-w-lg rounded-t-2xl sm:rounded-2xl bg-white shadow-xl max-h-[90vh] overflow-y-auto`. Bottom-sheet on mobile, centered card on `sm:` and up.
- Inner padding: `p-5 sm:p-6`. Footer actions: `flex flex-col-reverse sm:flex-row sm:justify-end gap-2` so Save sits on top on mobile (thumb-reach) but right on desktop.

### Forms

- Field pairs: `grid grid-cols-1 sm:grid-cols-2 gap-4`. Don't try to keep 3+ columns on mobile.
- Multi-step / wide forms: section-card wrappers stay `rounded-2xl border border-slate-200/70`, but their inner grid stacks at `<sm`.

### Chip filters and tab rows

- Wrap and allow horizontal scroll: `flex gap-2 overflow-x-auto -mx-1 px-1 scrollbar-thin` (or `flex flex-wrap gap-2` if the set is small and stable). Chips themselves keep `whitespace-nowrap`.

### Sidebar / topbar

- `AppLayout` and `EmployeeLayout` already handle the drawer + topbar hamburger correctly. Don't reintroduce a fixed left-pad on `<lg` — the `lg:pl-16` / `lg:pl-[260px]` is gated correctly.
- Topbar page title may need truncation on mobile (`truncate`) when long.

## Open items / future cleanup

- `kpi-cards.tsx`, `attendance-overview.tsx`, `alerts-panel.tsx`, `recent-activity.tsx` are unused dashboard components left in the tree — delete on next cleanup pass if still orphaned.
- Tables on legacy pages may still use old `<th>` header class strings (`p-3 text-left text-xs font-medium text-slate-500`) — update to the uppercase-tracking pattern on next touch.
- `hover:bg-[#f8fafc]` in button contexts (outside tables) is still present in many files — leave unless we move to `hover:bg-slate-50/60` project-wide.

---
> Source: [kantecalebdesigns/sabihr](https://github.com/kantecalebdesigns/sabihr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
