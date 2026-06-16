## general

> - **Platform**: Jekyll static site generator (4.3.x)

# Mayonic Medical - Technical Architecture

## Stack & Deployment
- **Platform**: Jekyll static site generator (4.3.x)
- **Hosting**: Optimized for GitHub Pages deployment
- **Dependencies**: Minimal - only Jekyll and Minima theme
- **Build Process**: Simple `bundle exec jekyll build`

## Sustainable Architecture
- **Base Theme**: Minima with custom overrides (no forking)
- **Custom Components**:
  - Header with reusable dropdown navigation pattern
  - Enhanced footer with configurable information sections
  - Flexible content templates with anchor linking
- **Asset Strategy**: Single compiled CSS file, minimal vanilla JS

## Content Structure
- Markdown content files (.md) with YAML front matter
- Section-based content with anchor links (#section-id)
- Standard Jekyll _includes, _layouts organization
- Configurable navigation via _config.yml header_pages

## Design System
- CSS custom properties for theming (colors, typography, spacing)
- Mobile-first responsive design (breakpoints at 600px, 768px)
- Reusable component patterns (cards, grids, dropdowns)

## Design Assets Organization
- **/.design/**: Root folder for all design assets (excluded from Jekyll build)
  - **/inspiration/**: Reference materials and mood boards
    - **/color-schemes/**: Color palette screenshots and swatches
    - **/typography/**: Font samples and type hierarchies
    - **/layouts/**: Layout examples and screenshots
  - **/components/**: UI component examples and patterns
    - **/navigation/**: Navigation patterns and examples
    - **/cards/**: Card design examples
    - **/forms/**: Form styling examples
  - **/branding/**: Brand assets
    - **/logos/**: Logo files in various formats
    - **/icons/**: Icon sets and custom icons
  - **/images/**: Source images (high-res versions)
    - **/photos/**: Photography assets
    - **/illustrations/**: Illustration assets
  - **/styles/**: Style guide and CSS reference
    - **/variables/**: CSS variable definitions
    - **/mixins/**: SCSS mixins and functions

## Performance Optimization
- No JavaScript frameworks or heavy dependencies
- Vanilla JS for interactive elements
- Optimized for fast load times and accessibility

---
> Source: [Drew-Goddyn/mayonic-medical](https://github.com/Drew-Goddyn/mayonic-medical) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
