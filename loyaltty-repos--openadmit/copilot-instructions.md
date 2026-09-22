## content-only-text-updates

> Content-only updates for a specified Next.js page. Only change text string literals. No style, layout, structure, props (except text-bearing), or behavior changes.


# Content-only updates for a specific Next.js page

Use this rule when editing copy on a single Next.js page. The goal is to update text content only. Do not introduce any changes that affect layout, styling, component structure, or runtime behavior.

If a target page file is provided in the task, apply all edits strictly within that file. Use the provided copy verbatim; do not paraphrase or alter wording, punctuation, or casing.

## Scope

- Allowed:
  - Update or insert string literals that are user-facing text in JSX/TSX (only simple string literals, not template literals or interpolated strings).
  - Change text nodes between JSX tags, e.g., `<p>...</p>`, `<h1>...</h1>` (only direct text content, not JSX elements).
  - Update text-bearing props only: `alt`, `title`, `aria-label`, `placeholder`, `value` (when static and user-facing), and similar purely textual constants.
  - Update static constants that hold user-facing copy (e.g., `const HEADING = "..."`).
  - Add new string literals exactly as provided when the task specifies additional copy to be added.
  - Replace existing string literals with new string literals of different length (no structural changes).
  - Update user-facing string literals inside page-scoped local components referenced by the page (see Targeting the page for scope rules).

- Forbidden:
  - Any changes to `className`, `style`, Tailwind classes, CSS modules, or any styling.
  - Adding/removing/reordering JSX elements or components that affects layout/structure.
  - Changing component props other than the allowed text-bearing ones above.
  - Renaming variables, functions, components, or files.
  - Logic changes, conditional rendering, hooks usage, imports/exports, or data fetching.
  - Refactoring, formatting unrelated code, or re-wrapping text that changes structure.
  - Changing template literals, string interpolation, or dynamic text generation.
  - Modifying `children` props that contain JSX elements (only text content allowed).
  - Adding/removing HTML attributes, data attributes, or event handlers.
  - Changing whitespace, line breaks, or indentation that affects rendering.
  - Modifying TypeScript types, interfaces, or type annotations.
  - Changing default values, function parameters, or variable declarations.
  - Editing Next.js page or layout metadata/head in any form, including:
    - `export const metadata` objects.
    - `export async function generateMetadata` implementations.
    - `head.tsx` files or usage of `<Head>` components.
    - `<meta>`, `<title>`, `<link>`, and other head tags (and their attributes like `content`).

## Content handling rules

- Use the provided copy exactly as given. Do not summarize, rewrite, or correct grammar/spelling unless explicitly instructed.
- When replacing existing text, replace the entire string literal.
- When adding new content, insert it as new string literals where directed without altering surrounding structure.
- Do not split or merge paragraphs unless explicitly instructed; maintain existing markup boundaries.
- Do not modify string literals that contain dynamic content, variables, or expressions.
- Preserve exact whitespace and formatting within string literals unless the provided copy specifies otherwise.
- Do not convert between single quotes, double quotes, or template literals unless the provided copy uses a different format.

## Targeting the page

- This rule is meant to be applied manually to the specific page mentioned in the task. Example targets in this repository include files under `src/app/.../page.tsx` or component files used exclusively by that page.
- If a concrete file path is provided, limit edits to that file and its page-specific local components only. Do not modify shared components.
- Page-scoped local components:
  - Typically live alongside the page (e.g., `src/app/.../_components/...`).
  - Are imported by the target `page.tsx` and not imported elsewhere.
  - May be verified by checking project-wide imports; if unsure about exclusivity, do not edit the component and request confirmation.
- Shared components (do not edit): anything under `src/components/**`, UI kits, or components imported by multiple pages.
- When the page renders multiple section components, audit each referenced page-scoped component and update only their user-facing string literals under the same restrictions as the page.

## Verification checklist (must hold true for the edit):

1. Only string literals changed or added (no template literals, interpolation, or dynamic content).
2. No new imports/exports were introduced.
3. No JSX element or component structure changed.
4. No styles, classes, or layout props changed.
5. Provided copy inserted verbatim.
6. Only text nodes of JSX are modified.
7. No HTML attributes, data attributes, or event handlers added/removed.
8. No whitespace or formatting changes that affect rendering.
9. No TypeScript types, interfaces, or annotations modified.
10. No function parameters, default values, or variable declarations changed.
11. No edits to Next.js metadata/head (`metadata`, `generateMetadata`, `head.tsx`, `<Head>`, head tags).
12. If edits occurred in a component, the component is page-scoped (not shared) and only user-facing string literals were changed.

## Notes

- If the task requires changing multiple files, apply this rule only to the explicitly named page file(s). If unsure, ask for the exact page path before proceeding.

---
> Source: [loyaltty-repos/openadmit](https://github.com/loyaltty-repos/openadmit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
