## flosports-fullstack-assessment

> Always style UI using the shared FloSports SCSS palette and tokens.


# FloSports Palette-First Styling

- **Use shared tokens**: Always pull colors, spacing, typography, radii, and breakpoints from the central SCSS variables file (`apps/ui/src/_variables.scss`) instead of hardcoding values.
  - **Colors**: Prefer `$flo-red`, `$flo-black`, `$color-bg-*`, `$color-text-*`, `$color-border*`, `$color-status-*`, `$color-stream-*`, etc.
  - **Spacing**: Use `$spacing-*` for margins/padding instead of raw `px`/`rem` unless a new token is truly needed.
  - **Typography**: Use `$font-size-*` for font sizes.
  - **Radii & layout**: Use `$radius-*`, `$content-max-width`, `$breakpoint-md` where applicable.

- **Match existing look and feel**:
  - New UI elements (buttons, badges, tooltips, cards, panels, skeletons, etc.) should visually align with existing components in `app.scss` and feature SCSS files.
  - When adding a new pattern (e.g., tooltip, badge, chip), scan for a similar existing pattern and borrow its colors and elevation (background, border, shadow) before introducing anything new.

- **No ad-hoc colors**:
  - Avoid introducing inline hex colors or arbitrary `rgba()` values directly in component SCSS.
  - If a new color or shade is required, add it once to `_variables.scss` with a clear semantic name (e.g., `$color-tooltip-bg`, `$color-badge-warning-bg`) and then use that token.

- **SCSS only, no inline styles**:
  - Do not add inline `style=""` attributes in templates for theme/visual styling; always prefer class names backed by SCSS that use the shared tokens.
  - Layout tweaks (flex/grid, gaps) can use SCSS utilities or component-specific classes, still driven by the spacing tokens.

- **When touching styles**:
  - If you notice existing hardcoded values that clearly duplicate palette tokens (e.g., `#ff140f` instead of `$flo-red`), prefer refactoring them to use the shared token while you are in that area, as long as it’s safe and scoped.
  - Keep new tokens and class names semantic (what they represent, not how they look), e.g. `.tooltip`, `.event-card__meta`, `.stream-health--excellent`, not `.red-text` or `.small-gray`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alienfacepalm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
