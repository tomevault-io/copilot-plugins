## flosports-fullstack-assessment

> Use SCSS for all styling; no Tailwind or other utility-first frameworks.


# SCSS-Only Styling

- **Use SCSS for all styles** (per PRD: "Use SCSS for styling").
  - Prefer component-level SCSS files (`styleUrls: ['./component.scss']` or `styleUrl`).
  - Use a global styles file (e.g. `styles.scss`) for resets, variables, and shared utilities.

- **Best practices**:
  - **Variables**: Define colors, spacing, typography, and breakpoints in a shared `_variables.scss` or in the global styles file; use them in components.
  - **BEM-style naming**: Prefer block__element--modifier (e.g. `.event-card`, `.event-card__title`, `.event-card--live`) for clarity and avoid deep nesting.
  - **No inline styles** for layout/theme; use classes and SCSS.
  - **Scoping**: Rely on Angular’s view encapsulation; avoid global class name collisions by using a consistent prefix or BEM block per feature.

- **Do not introduce**:
  - Tailwind or other utility-first CSS frameworks.
  - CSS-in-JS or third-party UI/theming libraries for styling (component libraries are still disallowed by the assessment).

- **When adding or changing styles**:
  - Add or update SCSS in the component’s style file or in the global stylesheet.
  - Reuse variables and mixins from the global layer instead of hardcoding colors or spacing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alienfacepalm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alienfacepalm)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
