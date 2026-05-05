## design-elements-commonality

> RBIP-specific UI constraints. Applied when building dashboard components.


# RBIP UI Constraints

## Component Library

| Element | Rule |
|---------|------|
| Icons | lucide-react only (no emojis) |
| Select | Radix UI `@/components/ui/select` (never native `<select>`) |
| Buttons - Selection | `bg-blue-600 text-white` active / `bg-gray-100 text-gray-700` idle |
| Buttons - Action | Semantic variants: `default`, `outline`, `destructive` |

## Layout Rules

- Max nesting: 2 levels (page → section/card)
- Grouping: use `space-y-6`, `divide-y`, `gap-4` (not nested borders)
- Dialog widths: `max-w-lg` default, `max-w-2xl` for tables/wide forms
- Text: allow wrapping (never `whitespace-nowrap` unless icon/button)
- **Flat over “AI box stacks”:** prefer one outer frame + `border-t` / `divide-y` + spacing between sections. Avoid nesting bordered cards for every block (status vs list vs actions). Use a **thin accent** (e.g. `border-l-2`) on text when emphasis is needed, not a second card.

## Slot labels (density)

- **Tight / lane / chip:** use **slot index** (`Slot 1` … `Slot 4` or `PCA · Slot N`) when horizontal space is small.
- **Primary / wide detail:** show the **time interval** for that slot (e.g. `0900-1030`); reuse `@/lib/utils/slotHelpers` (`getSlotTime`, `formatTimeRange`) so intervals match the rest of allocation UI.

## Visual Tokens

| Element | Style |
|---------|-------|
| Tooltips | `bg-white border border-border shadow-md` |
| Cards | Self-contained units only (not wrappers for every sub-element) |
| Elevation | Single subtle shadow level max |
| Borders | Use `border-border` or `divide-y`, not harsh solid borders |

## Step 3 — Raised target vs Extra after needs (semantic colors)

Use **emerald** vs **violet** consistently for these two meanings everywhere (badges, inline text, table hints, Step 3.1 copy above the card legend). **Do not** use Tailwind `purple-*` for **Extra after needs** — unify on `violet-*` so it matches Step 3.4 outline badges (`STEP34_EXTRA_AFTER_NEEDS_BADGE_CLASS`).

| Meaning | Hue | Typical tokens |
|---------|-----|----------------|
| **Raised target (shared spare)** — Step 2→3 rounding slack shared in the pool | Emerald | Badges: `border-emerald-600/85 bg-emerald-50 text-emerald-950` (see `STEP34_RAISED_TARGET_BADGE_CLASS`). Flat copy: `text-emerald-800 dark:text-emerald-200` (see `STEP31_RAISED_TARGET_TEXT_CLASS` in `FloatingPCAConfigDialogV2`). |
| **Extra after needs** — optional Step 3.4 slot(s) after basic floating need is met | Violet | Badges: `border-violet-600/85 bg-violet-50 text-violet-950` (`STEP34_EXTRA_AFTER_NEEDS_BADGE_CLASS`). Flat copy: `text-violet-800 dark:text-violet-200` (`STEP31_EXTRA_AFTER_NEEDS_TEXT_CLASS`). Small labels: `text-violet-700 dark:text-violet-300`. |

**Step 3.1 layout:** Keep scarcity preview (amber) for shortage only. Place **Raised target** literacy and **Extra after needs** preview in the same flat `space-y-2` block **above** “What the card numbers mean” — no nested card-on-card for those two lines.

## Banner Panel Design

For inline informational banners and confirmation panels:

### Container
- **Shape**: `rounded-xl` (not `rounded-md`)
- **Background**: `bg-{color}-50/40` (translucent tint, e.g., `bg-blue-50/40`)
- **Border**: `border-{color}-100/60` (subtle translucent, e.g., `border-blue-100/60`)
- **Shadow**: `shadow-sm` (single subtle elevation)
- **Padding**: `p-3` (compact inner spacing)
- **Width**: `w-full max-w-2xl` (constrained, not full-bleed)
- **Margin**: `mt-3` or context-appropriate

### Inner Content Layout
- Wrap content in inner div: `flex items-start gap-2 text-sm text-{color}-900`
- Icon: color-matched (e.g., `text-blue-600`) with `mt-0.5` for alignment
- Text: standard body text, allows wrapping

## Wide Viewport — Proximity & Eye-Tracking

Controls and their associated metadata (labels, status values, summaries) must stay in the same visual cluster. On wide viewports `justify-between` stretches related items to opposite ends of a row, forcing excessive horizontal eye-tracking. Default to left-aligned clusters; only push content to opposite ends when the two sides are genuinely independent.

| Situation | Wrong | Right |
|-----------|-------|-------|
| Control row (button + status value) | `flex justify-between` | `flex gap-3` (left-aligned) |
| Info row (label + summary text) | `flex justify-between` | `flex gap-4` (left-aligned) |
| Review list (name + FTE) on wide screen | Single full-width column | `lg:grid lg:grid-cols-2 lg:gap-px lg:bg-border` |
| Collapsible header summary | Mode label alone, pushed right | Mode label + inline preview, grouped left of chevron |

- ❌ `justify-between` to separate a value from the control it belongs to
- ❌ Full-width single-column lists on widescreen when a 2-col grid would halve eye-travel
- ❌ Metadata (FTE, slot summary, derived result) floating far right while its controls are on the left
- ❌ `flex-1` spacers to fill dead space between related items

## Anti-Patterns (Never Do)

- ❌ Nested `<Card>` beyond 2 levels
- ❌ Native `<select>` in dialogs
- ❌ Emoji icons (ℹ️ ⚠️ ✓)
- ❌ `max-w-4xl`+ dialogs without justification
- ❌ 3+ nested border levels
- ❌ Multiple accent colors (except the fixed **emerald vs violet** pair for Raised target vs Extra after needs above)
- ❌ Different hues for different surfaces

## Quick Reference: Spacing Scale

| Context | Value |
|---------|-------|
| Icon gaps | `gap-1`, `gap-2` |
| Component internal | `p-3`, `p-4` |
| Section gaps | `space-y-6`, `gap-6` |
| Major separation | `space-y-8` |

---
> Source: [alvin19921008-arch/RBIP-web-app](https://github.com/alvin19921008-arch/RBIP-web-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
