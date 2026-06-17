## co2

> This project is a focused Next.js App Router recreation of the "CO2 is no joke" paper-style layout. Most work here is either visual parity work or careful edits to the small set of page/card primitives that render the page.

# CO2 Paper Implementation

This project is a focused Next.js App Router recreation of the "CO2 is no joke" paper-style layout. Most work here is either visual parity work or careful edits to the small set of page/card primitives that render the page.

## Project structure
- `app/page.tsx` renders the single public page with inline article content and shared frame/card components.
- `components/` contains the full UI surface for this project: frame layout, section/place cards, accordion, bullets, citation arrow, formula, graph, hero image, and shared types.
- `app/globals.css` is the source of truth for the paper-themed design tokens, local font registration, and global element defaults.

## Project conventions
- Preserve the existing paper aesthetic: lime highlight, thin borders, hard shadows, mono typography, and compact vertical rhythm.
- Prefer extending the existing `components` primitives over introducing new abstractions.
- Keep content changes in `app/page.tsx` unless the task clearly calls for a reusable primitive.
- Treat `app/globals.css` theme tokens as the shared palette and typography contract. Promote reusable values there before scattering repeated literals.
- Match the current implementation style: simple typed props, small helper components in-file when local, and minimal indirection.
- Keep the page as a server component by default. Add `"use client"` only where interactivity is required, as in the accordion.

## Code style and conventions
- Simplicity first. Start with the simplest implementation then layer in complexity as needed.
- Paper & Figma MCP if provided, is your source of truth, liberally use it to check for visual parity.
- When using Paper MCP for recreation or parity work, read the repo-local skill at `.codex/skills/paper-parity/SKILL.md` first.
- Create semantic tokens from `@theme` in `app/globals.css` only when the style is reusable across multiple components.
- Avoid unnecessary client components, hooks, or state. This repo only needs them for genuinely interactive UI.
- Avoid arbitrary one-off spacing and sizing values unless the design requires them.
- If arbitrary values are dictated by Figma or Paper MCP, use the nearest Tailwind scale value unless exact parity matters.
- Arbitrary colors are acceptable temporarily; promote them to `@theme` tokens once they are reused.
- Keep copy edits and layout edits separate when possible. This makes visual regressions easier to trace.

## Hard rules
- Changes which aren't minor need to be verified by inspecting the DOM, see instructions below.

## Understand the DOM
For structural styling, complex DOM changes, or CSS/layout debugging, inspect the rendered DOM first. The rendered DOM is the source of truth. Do not infer selector paths from Next.js/React source alone when the change depends on parent/child relationships in the final DOM.

This includes:
- parent-driven styling like `*:` selectors, arbitrary selector variants, descendant/child combinators, and group/peer patterns
- Next.js wrapper behavior, hydration boundaries, and generated markup that may differ from the component tree
- React component boundaries where the source tree may not match the final DOM tree

Use the following workflow:
- **Inspect before editing** — Use `agent-browser eval` to run `getComputedStyle()` on the target element and its parent chain for relevant properties. Dump `outerHTML` to see the actual rendered markup and any injected inline styles or wrapper elements.
- **Identify the winning rule** — Before writing overrides, determine what's currently winning the cascade (inline styles, authored selectors, utility classes, browser defaults). Know the specificity you're fighting.
- **Verify after each change** — Re-run `getComputedStyle()` to confirm the target property changed on the actual visible element, not just a wrapper. Screenshots show *what's wrong*; computed styles show *why*.

---
> Source: [GLaDO8/co2](https://github.com/GLaDO8/co2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
