## maglev

> Maglev is a React-based component library and documentation system. It provides a set of reusable UI components and an integrated documentation platform to demonstrate their usage and styling.

# Maglev Library

Maglev is a React-based component library and documentation system. It provides a set of reusable UI components and an integrated documentation platform to demonstrate their usage and styling.

## Project Overview

- **Core Technologies:** React 17, TypeScript, Sass (SCSS), React Router 6.
- **Architecture:** 
  - `src/components/`: Contains the reusable UI components (e.g., MLButton, MLTabs).
  - `src/docs/`: Contains the documentation engine and demo pages.
  - `src/appearence/`: Global styles, color definitions, and theme configurations.
- **Theming:** Built-in support for light and dark themes using CSS variables and the `data-theme` attribute on the document root.

## Building and Running

The project is bootstrapped with Create React App.

- **Start Development Server:**
  ```bash
  npm start
  ```
- **Build for Production:**
  ```bash
  npm run build
  ```
- **Run Tests:**
  ```bash
  npm test
  ```
- **Linting:**
  ```bash
  npm run lint       # TypeScript linting
  npm run stylelint  # SCSS linting
  ```
- **Deployment:**
  ```bash
  npm run deploy     # Deploys to GCP App Engine (maglev-312521)
  ```

## Development Conventions

### Styling and Theming
- **SCSS Modules:** Use CSS Modules for component-specific styles (`ComponentName.module.scss`) to ensure style encapsulation.
- **CSS Variables:** Use the predefined CSS variables in `src/appearence/` for colors and theme-specific values. Do not hardcode hex values in component styles.
- **Themes:** Theming is handled via `src/appearence/light-theme.scss` and `src/appearence/dark-theme.scss`. Use `var(--variable-name)` to reference these.

### Component Structure
- Each component should reside in its own folder within `src/components/`.
- Include the component file (`.tsx`) and its corresponding style module (`.module.scss`).

### Adding Documentation
- To add a new component to the documentation:
  1. Create a demo page in `src/docs/pages/[component-name]/`.
  2. Register the component in `src/docs/docs-content.ts`.
  3. Add the route in `src/App.tsx`.

### Testing
- Place test files alongside the components they test, or in the `src/` directory following the `*.test.tsx` naming convention.
- Use React Testing Library for component tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/r-burkhardt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/r-burkhardt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
