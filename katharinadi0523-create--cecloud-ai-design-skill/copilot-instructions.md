## cecloud-ai-design-skill

> >-


# CeCloud enterprise AI design system

## Intent

Polished enterprise-tech look with a strong Chinese corporate identity: clean, authoritative, bright blue accents, generous white space, soft shadows, and high-impact hero imagery for scale, innovation, and trust. Premium but approachable; light shell around AI-forward promotional content.

## Color tokens

Map to CSS variables or Tailwind `theme.extend.colors` when implementing.

| Role | Hex | Usage |
|------|-----|--------|
| Primary / accent | `#2773ff` | Primary buttons, active states, highlights, section titles, key icons |
| Page / card surface | `#f8f9fb` | Main canvas, airy card backgrounds (generic screens) |
| Plaza main column (灰蓝) | `#e8f0fb` | **Application / capability plazas only**—full content column including layout padding; see **Plaza shell** in [plaza-page.md](references/page-types/plaza-page.md) |
| Plaza scroll tint (gradient) | `#f2f7fd` → `#e8f0fb` → `#e4edf8` | Optional **vertical** gradient on the **scroll wrapper** so the interior is not a flat gray slab |
| Body / nav text | `#5a6779` | Secondary hierarchy, readable UI copy |
| Strong headings | `#000000` | Hero and prominent card titles (use selectively) |
| Secondary blue | `#2f5fbf` | Title accents, formal corporate emphasis |
| Light blue tint | `#dbe7f4` | Shadows, borders, subtle surfaces |
| White | `#ffffff` | Nav bars, cards, buttons; text on dark imagery |
| Warm accent | `#ffb24a` | Hero light trails, graphic highlights (contrast only) |
| Cyan / teal (gradient family) | `#4fd6ff`, `#35c9f0` (and similar) | Decorative gradients, product icons, streaks |
| Deep navy / hero dark | `#0b1020` (and similar) | Hero depth and contrast |

## Typography

- **Stack**: PingFang SC first, then a modern sans-serif Chinese UI fallback stack.
- **Headlines**: Bold, compact; tight line height; no decorative styling. Sizes: H1 ~32px, H2 ~24px, H3 ~20px, H4 ~18px.
- **Body**: PingFang SC (lighter weight acceptable), ~16px; restrained, informational.
- **Labels / captions**: PingFang SC ~14px, regular; muted gray-blue (`#5a6779` family); functional, not stylized.

## Components

- **Primary button**: Pill shape, solid `#2773ff`, white text, no visible border or heavy shadow.
- **Secondary button**: White fill, blue outline, slightly softer rounding than primary if both appear together.
- **Link actions**: Plain text, no container; for nav and utilities.
- **Cards**: Very light background (`#f8f9fb` / white), no hard border, rounded corners, subtle diffuse shadow; generous padding; minimal noise.
- **Inputs**: Minimal; prefer light separators and muted placeholders over heavy boxed borders; header search uses pale dividers and simple icons.

## Spacing and elevation

- Rhythm from **6px** up to **~70px** in clear steps; wide separation around heroes and card grids.
- Shadows: **light and diffuse**—lift cards and floating panels only; avoid harsh depth.

## Do / don’t

**Do**

- Off-white or white base with **strong blue** for primary actions and focal points.
- Chinese UI–friendly type: **bold headlines**, lighter body.
- Soft shadows, rounded corners, **spacious** layouts.
- Large, high-contrast **hero imagery or gradients** for innovation / scale where the page type calls for it.

**Don’t**

- Dense borders, heavy outlines, or busy separators that feel rigid.
- Unrelated loud colors or decorative type that fights the brand blue and corporate tone.

## Page-type references

| Task | Specification file |
|------|---------------------|
| App shell, `html`/`body`, Arco semantic tokens | [references/shell-and-tokens.md](references/shell-and-tokens.md) |
| Plaza / application-square hub (应用广场) | [references/page-types/plaza-page.md](references/page-types/plaza-page.md) |
| List / management table (列表页) | [references/page-types/list-page.md](references/page-types/list-page.md) |
| Form / create–config (表单页) | [references/page-types/form-page.md](references/page-types/form-page.md) |

Merge **this file** (product palette + principles) with **shell-and-tokens** always; add **plaza-page**, **list-page**, and/or **form-page** depending on the screen pattern.

## Implementation note (Nexus / Next.js)

Prefer design tokens (CSS variables or Tailwind theme) over ad-hoc colors. Marketing heroes may be full-bleed where layout allows. Reserve orange and cyan gradients for **accent** moments, not primary chrome.

**Form docked footer in this repo:** long forms under **`app/(dashboard)/layout.tsx`** (`p-6` + 220px sidebar) should follow **form-page.md →「Docked action bar (workspace shell)」** when the action row must **flush to the viewport bottom** and **span the main column** — `sticky` alone inside the padded shell is usually wrong.

### Skills / capability plaza (`/skills-hub`, `/skills`)

When implementing a **plaza** in this repo, align with **Plaza shell** in [plaza-page.md](references/page-types/plaza-page.md):

1. **Layout** (`app/(dashboard)/layout.tsx`): for **hub-only** pathnames, set the **flex row under the header**, **`main`**, and the **scroll container** so the **gutter** (`p-6`) sits on **`#e8f0fb`**, not on default `slate-50` white.
2. **Scroll wrapper**: add the **documented linear-gradient** so the scroll area has a slight **灰蓝** depth; avoid stacking an opaque `#f8f9fb` page root on top (use **`bg-transparent`** on the plaza page component).
3. **Scope**: do **not** reuse the plaza shell for **`/skills-management`**—keep the default shell so list admin stays visually distinct.

Utility **`.skills-plaza-canvas`** and **`--cc-surface-tint`** in `app/globals.css` should stay in sync with **`--plaza-shelf` / `#e8f0fb`** unless the product token table is extended in `tailwind` theme.

---
> Source: [katharinadi0523-create/cecloud-ai-design_SKILL](https://github.com/katharinadi0523-create/cecloud-ai-design_SKILL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
