## match-reference-design

> Match reference images and designs exactly; do not substitute variants


# Match Reference Design Exactly

When the user provides a reference image, mockup, or design:

1. **Implement to match it exactly.** Do not substitute different variants (e.g. solid vs regular icons, filled vs outline shapes, different icon sets) unless the user explicitly asks for a change.

2. **Re-check the reference** when implementing details: icon style (regular/outline vs solid), colors, alignment, spacing, typography (weight, case, size). If the reference specifies "regular" or "outline," use that; do not default to solid or filled.

3. **When in doubt, ask** rather than guessing. If the reference is ambiguous, confirm before implementing.

## Theme tokens and new primitives

1. When editing SCSS/CSS in this theme:
   - **Only use existing variables, mixins, and functions** defined in:
     - `library/scss/partials/_variables.scss`
     - `library/scss/partials/_typography.scss`
   - Do **NOT** invent new variable names (e.g. `$radius-xl`, `$new-color`) or new font stacks unless the user explicitly asks for them by name.

2. Before using any SCSS variable:
   - The assistant must **confirm it exists** in `_variables.scss` or `_typography.scss`.
   - If it does not exist, the assistant must either:
     - Reuse an existing variable, **or**
     - Propose adding a new token in a message and wait for explicit user approval before changing any files.

3. **Reference work still uses tokens.** Matching a comp/mockup does **not** allow new palette variables (e.g. `$stories-*`), comp-only hex blocks, or one-off radius names. Map the reference to the **closest existing** `$primary-*`, `$secondary-*`, `$accent-*`, `$grey-*`, `$radius-*`, etc.

4. Layout and typography:
   - Prefer **existing classes and mixins** (e.g. `h2.hero`, `$header-font`, `$body-font`, `$spacing-*`, `$radius-*`).
   - Do **not** introduce new global layout primitives or templates (e.g. new single templates, new layout systems) unless the user explicitly requests them.

---
> Source: [warofthesun/tectn-theme](https://github.com/warofthesun/tectn-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
