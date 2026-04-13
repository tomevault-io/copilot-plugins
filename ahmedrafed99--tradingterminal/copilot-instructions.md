## tradingterminal

> This project uses feature-folder READMEs as the primary source of context. Before modifying or discussing any feature, **read `README.md` (root)** first — it contains a folder map and quick lookup table that points to the correct feature folder.

# CLAUDE.md

## Documentation-First Workflow

This project uses feature-folder READMEs as the primary source of context. Before modifying or discussing any feature, **read `README.md` (root)** first — it contains a folder map and quick lookup table that points to the correct feature folder.

### When the user asks about a feature

1. Open `README.md` at the project root
2. Use the **Quick Lookup** table to find the relevant feature folder
3. Read that folder's `README.md` for full context
4. For frontend implementation details (store slices, service signatures, component internals), also read `frontend/README.md`

### After making code changes

If a code change affects behavior documented in any feature README, prompt the user:

> "This change affects [feature]. Want me to update `[folder]/README.md` to reflect the new behavior?"

Do not silently skip documentation updates — the READMEs are how future sessions recover context.

## Visual Design System

This project has a strict set of design tokens. **Do not invent new color values, opacities, or border radii.** Use only the values in the table at `README.md → Design Tokens`. When in doubt, match the nearest existing component.

### Rules

1. **Transitions everywhere** — Every interactive state change (hover, focus, active, open/close) must be animated. Use `transition-colors` (Tailwind) or `transition: background 0.15s, color 0.15s` (inline). Dropdowns and popovers should fade/slide in, never pop instantly.
2. **Borders are always `#2a2e39`** — No `#222`, no `#333`, no `rgba` border hacks. One border color.
3. **Disabled state is always `disabled:opacity-50`** — Not 30, not 40. One value.
4. **Modal backdrop is `bg-black/60`** — Consistent dimming across all modals.
5. **Modal panel background is `bg-[#1e222d]`** — Not `bg-black`. Panels float above the page.
6. **Input/control background is `bg-[#111]`** — Not `#0a0a0a`, not `#000`.
7. **Hover backgrounds** — Use `hover:bg-[#1e222d]` for list rows/dropdowns, `hover:bg-[#363a45]` for toolbar icon buttons. Never use JS `onMouseEnter`/`onMouseLeave` for hover styling — use Tailwind `hover:` classes.
8. **Section labels** — `text-[10px] uppercase tracking-wider text-[#787b86]` for all panel section headers.
9. **No new colors** — If a design needs a color not in the token table, ask first. Don't add one-off hex values.

## Tailwind Gotcha

Tailwind JIT sometimes fails to generate utility classes (especially spacing like `px-*`, `py-*`, `gap-*`) when they haven't been used elsewhere in the project. If a Tailwind class doesn't take effect, **use inline `style={{ ... }}` instead**. This has been confirmed multiple times in this codebase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahmedrafed99)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ahmedrafed99)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
