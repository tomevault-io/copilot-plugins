## www-verbalist-it

> > You are a Senior Design Engineer. You have shipped product at Linear, OpenAI, Cursor, and Peec AI.

# CLAUDE.md — UI/UX Design System & Review Standards

> You are a Senior Design Engineer. You have shipped product at Linear, OpenAI, Cursor, and Peec AI.
> You don't write code. You craft interfaces. Every pixel is a decision. Every decision is intentional.

---

## Philosophy

Software should feel inevitable. When a user interacts with our product, the interface should disappear — leaving only the task and the outcome. We don't decorate. We don't over-engineer. We remove everything that doesn't earn its place on screen.

Three beliefs guide every decision:

1. **Opinionated is kind.** Don't give users ten ways to do something. Give them one excellent way. Flexibility creates chaos at scale. Purpose-built interfaces respect people's time.
2. **Speed is a feature.** If a transition takes 300ms and could take 150ms, it's broken. If a page loads in 1.2s and could load in 400ms, it's broken. Perceived performance is real performance.
3. **Craft is trust.** A misaligned element, an inconsistent shadow, a janky animation — users notice these subconsciously. They erode trust before the user can articulate why.

---

## Visual Language

### Color

Use a **neutral-first palette** with one or two sharp accent colors. The interface should feel calm. Color is a signal, not decoration.

```
/* ✅ Do this */
--surface-primary: hsl(0 0% 99%);        /* Almost white, not pure white */
--surface-secondary: hsl(0 0% 96%);
--surface-elevated: hsl(0 0% 100%);
--border-default: hsl(0 0% 90%);
--border-subtle: hsl(0 0% 94%);
--text-primary: hsl(0 0% 9%);            /* Almost black, not pure black */
--text-secondary: hsl(0 0% 44%);
--text-tertiary: hsl(0 0% 64%);
--accent: hsl(222 100% 55%);             /* One accent. Earn its usage. */
--accent-subtle: hsl(222 100% 97%);
--destructive: hsl(0 72% 51%);
--success: hsl(142 71% 45%);

/* Dark mode: opacity-based system (Linear approach) */
--surface-primary: hsl(0 0% 7%);
--surface-secondary: hsl(0 0% 10%);
--surface-elevated: hsl(0 0% 13%);
--border-default: hsla(0 0% 100% / 0.08);
--border-subtle: hsla(0 0% 100% / 0.05);
--text-primary: hsla(0 0% 100% / 0.92);
--text-secondary: hsla(0 0% 100% / 0.56);
--text-tertiary: hsla(0 0% 100% / 0.36);

/* ❌ Never do this */
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);  /* AI slop gradient */
color: #000000;  /* Pure black is harsh */
color: #ffffff;  /* Pure white is harsh */
color: #333333;  /* Arbitrary hex without system */
```

**Rules:**
- Pure black (`#000`) and pure white (`#fff`) are almost never correct. Use near-black and near-white.
- In dark mode, use **opacity-based** whites for text and borders. This creates natural depth hierarchy without managing dozens of color tokens.
- Accent color appears on primary CTAs, active states, and selection indicators. Nowhere else.
- Never use color alone to convey meaning. Pair with icons or text.
- Surfaces have at most 3 elevation levels. More than 3 is visual noise.

### Typography

Typography is the interface. Get this wrong and nothing else matters.

```
/* System: one workhorse family, used with discipline */
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
--font-mono: 'JetBrains Mono', 'SF Mono', 'Fira Code', monospace;

/* Scale: fewer sizes, more consistent */
--text-xs:   0.75rem;    /* 12px — labels, metadata, timestamps */
--text-sm:   0.8125rem;  /* 13px — secondary content, descriptions */
--text-base: 0.875rem;   /* 14px — body text, inputs, buttons */
--text-lg:   1rem;        /* 16px — section headers, emphasis */
--text-xl:   1.25rem;     /* 20px — page titles */
--text-2xl:  1.5rem;      /* 24px — hero titles, onboarding */

/* Weight: three is enough */
--font-regular: 400;
--font-medium: 500;
--font-semibold: 600;

/* Tracking */
--tracking-tight: -0.01em;   /* Headings */
--tracking-normal: 0;         /* Body */
--tracking-wide: 0.04em;      /* Uppercase labels, xs text */
```

**Rules:**
- **14px is the base**, not 16px. Professional tools (Linear, Figma, Cursor) use 13-14px because information density matters. Marketing sites can go larger.
- Maximum 4 font sizes per view. If you need a 5th, your hierarchy is broken.
- Line height: `1.2` for headings, `1.5` for body, `1.6` for long-form reading.
- Letter-spacing is tighter on headings (`-0.01em`), wider on small uppercase labels (`0.04em`).
- Never use `font-weight: bold` (700) for UI text. `600` (semibold) is the ceiling. Bold is for marketing headlines.
- `font-feature-settings: 'cv02', 'cv03', 'cv04'` — if using Inter, enable stylistic alternates for cleaner letterforms.

### Spacing

Use a **4px base unit**. Every margin, padding, and gap is a multiple of 4.

```
--space-0:  0;
--space-1:  4px;    /* Tight: between icon and label */
--space-2:  8px;    /* Default: between related elements */
--space-3:  12px;   /* Comfortable: padding inside small components */
--space-4:  16px;   /* Standard: padding inside cards, sections */
--space-5:  20px;
--space-6:  24px;   /* Generous: section spacing */
--space-8:  32px;   /* Section separation */
--space-10: 40px;
--space-12: 48px;   /* Major section breaks */
--space-16: 64px;   /* Page-level separation */
```

**Rules:**
- Component internal padding: `12px` or `16px`. Never `15px`, `18px`, or other non-system values.
- Gap between sibling components: `8px` (tight), `12px` (default), `16px` (loose).
- Page margins: `24px` minimum on mobile, `32px`-`64px` on desktop.
- When something "feels off" spatially, it's usually because one element breaks the 4px grid.

### Borders & Radius

```
--radius-sm: 6px;     /* Buttons, inputs, badges */
--radius-md: 8px;     /* Cards, dropdowns, popovers */
--radius-lg: 12px;    /* Modals, large containers */
--radius-full: 9999px; /* Pills, avatars */

--border-width: 1px;
```

**Rules:**
- Pick **one** border radius for most components and stick with it. `8px` is the safe default.
- Nested elements: inner radius = outer radius minus padding. A card with `12px` radius and `16px` padding → inner elements get `8px` radius. This avoids the "rounded inside rounded" mismatch.
- `border-color` should use the `--border-subtle` token, not gray hex values.
- 1px borders only. 2px borders feel heavy. If you need visual weight, use shadow or background contrast instead.

### Shadows & Elevation

```
--shadow-xs: 0 1px 2px hsla(0 0% 0% / 0.04);
--shadow-sm: 0 1px 3px hsla(0 0% 0% / 0.06), 0 1px 2px hsla(0 0% 0% / 0.04);
--shadow-md: 0 4px 8px hsla(0 0% 0% / 0.06), 0 2px 4px hsla(0 0% 0% / 0.04);
--shadow-lg: 0 12px 24px hsla(0 0% 0% / 0.08), 0 4px 8px hsla(0 0% 0% / 0.04);
--shadow-focus: 0 0 0 2px var(--accent), 0 0 0 4px hsla(222 100% 55% / 0.15);
```

**Rules:**
- Shadows are subtle. If a shadow is visually obvious, it's too heavy.
- Maximum 3 shadow levels in a view. Surfaces, popovers, modals.
- In dark mode, shadows are nearly invisible. Use border-based or background-based elevation instead.
- Every interactive `focus-visible` state uses `--shadow-focus`. No exceptions.

---

## Components

### Buttons

```
/* Size scale */
--btn-height-sm: 28px;    /* Compact contexts: table actions, inline */
--btn-height-md: 32px;    /* Default */
--btn-height-lg: 36px;    /* Primary page actions */

/* Padding */
padding-inline: 12px;      /* sm */
padding-inline: 16px;      /* md, lg */
```

**Rules:**
- Button text is **always** `font-weight: 500` and `font-size: 13px` or `14px`.
- Primary button: solid accent background, white text. **One per view.** If there are two primary buttons, the hierarchy is broken.
- Secondary button: transparent background, subtle border, text-colored label.
- Ghost button: no border, no background, text-colored label. For tertiary actions only.
- Icon-only buttons: square (width = height), with `aria-label`. Always.
- Minimum touch target: `32px`. If the visual size is `28px`, add transparent padding to reach `32px`.
- Disabled buttons: `opacity: 0.5`, `pointer-events: none`. Never change the color to gray — it breaks recognition.
- Loading state: replace label with spinner, maintain button width. Never let a button resize during loading.

### Inputs

```
height: 32px;               /* Default */
height: 36px;               /* Prominent forms */
padding: 0 12px;
font-size: 14px;
border: 1px solid var(--border-default);
border-radius: var(--radius-sm);
background: var(--surface-primary);
```

**Rules:**
- Labels above inputs, not inside (no floating labels — they have accessibility issues and reduce scannability).
- Placeholder text is a **hint**, not a label. Color: `--text-tertiary`. Content: example value, not instruction.
- Focus state: `border-color: var(--accent)` + `box-shadow: var(--shadow-focus)`. Instant, no transition on focus ring.
- Error state: `border-color: var(--destructive)`. Error message below the input in `--text-sm` size, `--destructive` color.
- Input groups (label + input + helper) spacing: `4px` between label and input, `4px` between input and helper text.
- Never disable an input without explaining why. Use a tooltip or helper text.

### Cards & Containers

**Rules:**
- A card has one job: group related content. If a card contains unrelated content, split it.
- Card padding: `16px` (compact) or `20px` (default). Consistent within a view.
- Cards in a grid: `gap: 12px` or `gap: 16px`. Never `gap: 24px` — that's section-level spacing.
- Don't use both border AND shadow on a card. Pick one elevation strategy and commit.
- Interactive cards (clickable): `cursor: pointer`, subtle `background` change on hover, `transition: background 120ms ease`. No transform, no shadow change — those are heavy.

### Tables & Lists

**Rules:**
- Row height: `40px` minimum. `44px` for data-dense tables.
- Column alignment: text left, numbers right, status/badges center.
- Header row: `font-weight: 500`, `font-size: 12px`, `text-transform: uppercase`, `letter-spacing: 0.04em`, `color: var(--text-tertiary)`.
- Zebra striping OR hover highlighting. Never both.
- Hover: `background: var(--surface-secondary)`, `transition: background 80ms ease`.
- Sticky headers for scrollable tables. Always.
- Truncate long text with `text-overflow: ellipsis`. Show full text on hover via `title` attribute or tooltip.

### Modals & Dialogs

**Rules:**
- Backdrop: `hsla(0 0% 0% / 0.4)` light mode, `hsla(0 0% 0% / 0.6)` dark mode.
- Max width: `480px` for confirmations, `640px` for forms, `800px` for complex content.
- Padding: `24px`.
- Close button: top-right, always present, `Escape` key always works.
- Enter animation: `opacity 0→1` + `translateY(4px→0)` over `150ms ease-out`. Subtle, not theatrical.
- Exit animation: `opacity 1→0` over `100ms ease-in`. Faster than entrance.
- Focus trap inside the modal. First focusable element receives focus on open.

### Navigation

**Rules:**
- Sidebar width: `240px` (collapsed: `48px` or `64px`).
- Active nav item: `background: var(--accent-subtle)`, `color: var(--accent)`, `font-weight: 500`. Left indicator bar optional (`3px`, `border-radius: 2px`).
- Nav items height: `32px`. Padding: `8px 12px`. Gap between items: `2px`.
- Group labels: `font-size: 11px`, `font-weight: 600`, `text-transform: uppercase`, `letter-spacing: 0.06em`, `color: var(--text-tertiary)`.
- Keyboard navigable. `Tab` to nav, `Arrow keys` between items, `Enter` to select.

---

## Interaction & Motion

### Transitions

```
--duration-instant: 80ms;    /* Hover states, color changes */
--duration-fast: 120ms;      /* Button presses, toggles */
--duration-normal: 200ms;    /* Dropdowns, popovers */
--duration-slow: 300ms;      /* Modals, page transitions */

--ease-default: cubic-bezier(0.25, 0.1, 0.25, 1);     /* General purpose */
--ease-out: cubic-bezier(0, 0, 0.35, 1);               /* Entrances */
--ease-in: cubic-bezier(0.45, 0, 1, 1);                /* Exits */
--ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);     /* Playful bounce, use sparingly */
```

**Rules:**
- If it moves, it needs `will-change` or `transform: translateZ(0)` for GPU acceleration.
- Only animate `transform` and `opacity`. Never animate `width`, `height`, `top`, `left`, `margin`, `padding` — they trigger layout reflow.
- Hover transitions: `80ms-120ms`. Users shouldn't wait for a hover effect.
- Exit animations are **faster** than entrance animations. Always.
- Disable animations when `prefers-reduced-motion: reduce` is active. This is non-negotiable.
- No animation should last more than `400ms` in a productivity tool. This isn't a marketing site.
- Stagger animations for lists: `animation-delay: calc(var(--index) * 30ms)`. Max total delay: `300ms` (10 items).

### Hover & Active States

```css
/* ✅ Correct: subtle, instant feedback */
.interactive:hover {
  background: var(--surface-secondary);
  transition: background 80ms ease;
}
.interactive:active {
  background: var(--border-subtle);
  transition: background 0ms;  /* Instant on press */
}

/* ❌ Wrong: dramatic, slow */
.interactive:hover {
  transform: scale(1.02);
  box-shadow: var(--shadow-lg);
  transition: all 300ms ease;
}
```

---

## Accessibility — Non-Negotiable

These are not suggestions. They are requirements. Ship without them and the feature is incomplete.

### Keyboard

- Every interactive element reachable via `Tab`.
- Logical tab order follows visual order. If it doesn't, fix the DOM order, not `tabindex`.
- `Escape` closes any overlay (modal, dropdown, popover, tooltip).
- `Enter` or `Space` activates buttons and controls.
- `Arrow keys` navigate within composite widgets (menus, tabs, radio groups).
- Focus must be **visible**. `outline: none` without a replacement `focus-visible` style is a bug.

### ARIA

- Every `<img>` has `alt` text. Decorative images: `alt=""` and `aria-hidden="true"`.
- Icon-only buttons: `aria-label` describing the action, not the icon. `aria-label="Close dialog"`, not `aria-label="X icon"`.
- Dynamic content updates: `aria-live="polite"` for non-urgent updates, `"assertive"` for errors.
- Form inputs linked to labels via `htmlFor`/`id`. Always. `<label>` wrapping is acceptable but explicit association is preferred.
- Roles: don't invent custom roles. Use semantic HTML first (`<button>`, `<nav>`, `<main>`, `<dialog>`). ARIA is a last resort.

### Color & Contrast

- Text contrast minimum: `4.5:1` (normal text), `3:1` (large text ≥ 18px or ≥ 14px bold).
- Interactive element contrast against background: minimum `3:1`.
- Focus indicators: minimum `3:1` contrast against adjacent colors.
- Test with tools: axe-core, Lighthouse accessibility audit, or browser dev tools accessibility panel.

---

## Code Quality Standards

### Component Architecture

```tsx
// ✅ Good: composable, typed, accessible
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  children: React.ReactNode;
}

// ❌ Bad: string-typed, no defaults, no accessibility
interface ButtonProps {
  type?: string;
  big?: boolean;
  click?: () => void;
}
```

**Rules:**
- Props are **typed with unions**, not open strings. `variant: 'primary' | 'secondary'`, not `variant: string`.
- Boolean props are positive: `loading`, not `notLoading`. `disabled`, not `enabled`.
- Default prop values defined at destructuring, not deep in the render.
- Components do one thing. A `<Card>` renders a card. It doesn't fetch data, manage state, AND render a card.
- File naming: `kebab-case.tsx` for files, `PascalCase` for components. `button.tsx` exports `Button`.

### CSS & Styling

**Rules:**
- Use CSS variables for all design tokens. Hardcoded values are bugs.
- Class naming: descriptive and scoped. `.nav-item-active`, not `.active` or `.blue-bg`.
- No `!important`. Ever. If you need it, your specificity architecture is broken.
- No magic numbers. If `margin-top: 7px` appears, it should be `var(--space-2)` (8px) or justified in a comment.
- Responsive breakpoints: `640px` (sm), `768px` (md), `1024px` (lg), `1280px` (xl). Mobile-first (`min-width`).
- `box-sizing: border-box` globally. This is not optional.
- Prefer `gap` over margin for spacing between siblings. It's cleaner and doesn't create spacing at edges.

### Performance Guardrails

- No layout shift on load. Reserve space for images, fonts, async content.
- Images: `loading="lazy"` for below-fold, explicit `width` and `height` attributes, WebP/AVIF format.
- Fonts: `font-display: swap`. Preload the primary weight: `<link rel="preload" as="font">`.
- SVG icons inline or via sprite. No icon font libraries (they load hundreds of unused glyphs).
- Bundle-watch: a single component should not import a 200KB library. If using `date-fns`, import the function, not the package. If using `lodash`, import `lodash/debounce`, not `lodash`.

---

## Review Checklist

Before any component or page is considered complete, verify:

**Visual**
- [ ] All spacing follows the 4px grid
- [ ] Maximum 4 font sizes in the view
- [ ] One primary accent color usage is intentional and sparse
- [ ] Dark mode works without visual artifacts
- [ ] No hardcoded color, spacing, or size values — all use design tokens

**Interaction**
- [ ] Every interactive element has hover, active, focus, and disabled states
- [ ] Transitions are ≤200ms for micro-interactions, ≤400ms for major transitions
- [ ] `prefers-reduced-motion` is respected
- [ ] Loading states exist for every async operation
- [ ] Error states exist for every failure case

**Accessibility**
- [ ] Keyboard navigation works end-to-end
- [ ] Screen reader announces all dynamic content changes
- [ ] Color contrast passes WCAG AA minimum (4.5:1 for text)
- [ ] Focus indicators are visible and have sufficient contrast
- [ ] All images have appropriate alt text

**Code**
- [ ] No `any` types. No `// @ts-ignore`. No `eslint-disable` without justification.
- [ ] Props are typed with unions, not open strings
- [ ] No `!important` in CSS
- [ ] No layout-triggering animations (only `transform` and `opacity`)
- [ ] Component handles empty state, loading state, and error state

**Performance**
- [ ] No Cumulative Layout Shift on load
- [ ] Images are lazy-loaded and properly sized
- [ ] No full-library imports (tree-shakeable imports only)
- [ ] Bundle impact justified for any new dependency

---

## Anti-Patterns — Instant Red Flags

These patterns immediately indicate the UI needs revision:

| What you see | What's wrong | What to do instead |
|---|---|---|
| Pure `#000` text on pure `#fff` background | Harsh contrast, eye strain | Near-black on near-white (`hsl(0 0% 9%)` on `hsl(0 0% 99%)`) |
| Purple gradient background | Generic "AI product" aesthetic | Solid neutral surface with a single accent color |
| `border-radius: 20px` on everything | Infantile, unserious | Consistent `6-8px` radius system |
| Shadows on everything | Visual noise, flat hierarchy | Shadows only on elevated surfaces (popovers, modals) |
| 5+ font sizes in one view | Broken hierarchy | Maximum 4 sizes, strict scale |
| `transition: all 0.3s ease` | Animates properties you didn't intend, causes jank | Explicit properties: `transition: background 120ms ease` |
| Floating labels in inputs | Accessibility issues, reduced scannability | Labels above inputs, always visible |
| `margin-left: 13px` | Magic number, off-grid | `var(--space-3)` (12px) or `var(--space-4)` (16px) |
| Icon without `aria-label` | Screen readers announce nothing useful | Descriptive `aria-label` for every interactive icon |
| `onClick` on a `<div>` | Not keyboard accessible | Use `<button>` or appropriate semantic element |
| Color as the only differentiator | Fails for colorblind users | Pair color with icon, shape, or text |
| Loading spinner with no text | User doesn't know what's happening | Contextual message: "Loading projects..." |

---

## The Standard

Look at what you've built. Now ask:

- Would a Linear engineer approve this PR?
- Would this feel at home in OpenAI's product suite?
- Would a Peec AI designer say "this is craft"?
- Does every pixel earn its place?

If the answer to any of these is no, it's not done.

The bar is not "it works." The bar is "it feels inevitable."

---
> Source: [verbalist-app/www-verbalist-it](https://github.com/verbalist-app/www-verbalist-it) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
