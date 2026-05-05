## css-styling-architecture-rules

> **Stack**: Tailwind CSS + CSS Variables + shadcn/ui


# CSS & Styling Architecture Rules

**Stack**: Tailwind CSS + CSS Variables + shadcn/ui
**Audience**: Senior / Architecture-level developers

These rules define how styling is structured, scaled, and governed in this project. They intentionally restrict freedom to ensure visual consistency, themeability, and long-term maintainability.

---

## 1. Styling Philosophy

- Styling is **system design**, not decoration.
- Every visual decision must be:

  - Token-driven
  - Theme-aware
  - Predictable across surfaces

Avoid ad-hoc CSS, magic numbers, and one-off overrides.

---

## 2. Tailwind Is the Only Styling Interface

- Tailwind utility classes are the **primary and preferred** styling mechanism.
- No custom CSS files for component styling.
- Inline styles are forbidden except for:

  - Truly dynamic runtime values (e.g., measured dimensions)

If Tailwind cannot express a style, the design system is incomplete.

---

## 3. CSS Variables Are the Design Tokens

### Source of Truth

- All colors, radii, spacing scales, and semantic tokens **must originate from CSS variables**.
- Variables are defined at:

  - `:root` for base theme
  - `[data-theme]` or `.dark` for theme overrides

Example intent (illustrative):

- `--background`
- `--foreground`
- `--primary`
- `--radius`

Hardcoded hex values, rgba values, or raw HSL values in components are forbidden.

---

## 4. Tailwind Configuration Rules

- Tailwind `theme.extend` must reference **CSS variables**, not raw values.
- Do not redefine Tailwind defaults unless there is a system-wide reason.

Allowed:

- Semantic tokens (`primary`, `muted`, `destructive`)
- Layout primitives (container widths, breakpoints if justified)

Avoid introducing overlapping or redundant utilities.

---

## 5. shadcn/ui Is the Component Baseline

- shadcn components are:

  - The canonical UI primitives
  - Locally owned and modifiable

Rules:

- Do not wrap shadcn components just to restyle them
- Extend via:

  - Props
  - Variants
  - Class composition

Forking a shadcn component is acceptable **only** when behavior or structure must change.

---

## 6. Variants Over Conditionals

- Use `cva` / variant-based APIs for all component styling permutations.
- Conditional class concatenation (`if/else` Tailwind strings) should be minimal.

Every variant should map to a **design decision**, not a visual hack.

---

## 7. Layout vs Component Responsibility

### Layout Styling

- Page and layout components control:

  - Spacing
  - Grid / flex structure
  - Alignment

### Component Styling

- Components control:

  - Internal spacing
  - Visual identity
  - States (hover, focus, disabled)

Components must not assume where or how they are placed.

---

## 8. Responsiveness Strategy

- Mobile-first always.
- Breakpoints represent **design shifts**, not device targeting.

Rules:

- Do not sprinkle responsive classes without intent
- Avoid deeply nested breakpoint overrides

If responsiveness becomes complex, revisit the layout architecture.

---

## 9. Dark Mode and Theming

- Dark mode is a **first-class requirement**, not an afterthought.
- Themes are implemented via CSS variable overrides, not conditional classes.

Tailwind `dark:` utilities are allowed **only** when they map directly to token usage.

---

## 10. Animation and Transitions

- Prefer Tailwind utilities for transitions and animations.
- Custom keyframes must be:

  - Token-aligned
  - Reusable
  - Purpose-driven

Avoid gratuitous motion. Animation should communicate state change.

---

## 11. What Is Explicitly Forbidden

- Component-specific CSS files
- Random `bg-[#xxxxxx]` usage
- One-off spacing hacks (`mt-[37px]`)
- Styling driven by JS conditionals instead of variants
- Visual fixes that bypass the design token system

---

## 12. Review Bar for Styling Changes

Every styling change must answer:

- Which design token does this rely on?
- Does this survive theme changes?
- Does this scale across screens and contexts?
- Is this encoded as a variant or a one-off?

If the answer is unclear, the styling change is rejected.

---

## Final Principle

> The UI should feel inevitable, not accidental.

If styling decisions cannot be explained in system terms, they do not belong in the codebase.

---
> Source: [JaimeenMakavana/agent-vis](https://github.com/JaimeenMakavana/agent-vis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
