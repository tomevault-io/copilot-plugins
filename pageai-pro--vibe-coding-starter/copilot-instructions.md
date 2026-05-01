## project-structure

> This rule explains the folder structure of the project. Use this as a reference for navigating and understanding the codebase.


# Project Structure Guide

This rule explains the folder structure of the project. Use this as a reference for navigating and understanding the codebase.

## Root Directory

The root contains configuration files and the main project folders:
- [package.json](mdc:package.json): Project dependencies and scripts
- [tsconfig.json](mdc:tsconfig.json): TypeScript configuration
- [next.config.js](mdc:next.config.js): Next.js configuration
- [tailwind.config.js](mdc:tailwind.config.js): Tailwind CSS configuration

## Key Directories

### [components/](mdc:components)

Reusable React components, organized by feature:

- `landing/`: Landing page components
- `shared/`: Shared UI primitives (often Shadcn-based)
- `search/`: Search UI components
- `icons/`: SVG and icon components

### [app/](mdc:app)

Next.js App Router directory. Contains all route definitions and API endpoints:

- `page.tsx`: Main entry point
- `layout.tsx`: Root layout
- `not-found.tsx`: 404 page
- `seo.tsx`, `theme-providers.tsx`: SEO and theme context
- Subfolders for routes: `about/`, `api/`, `terms/`, `pricing/`, `privacy/`, etc.

### [scripts/](mdc:scripts)

Node scripts for build, RSS generation, and app info automation.

### [css/](mdc:css)

Global and syntax highlighting CSS files.

### [lib/](mdc:lib)

Utility functions and helpers.

### [components/shared/](mdc:components/shared)

**Shared Components**: Common UI and logic used across the app, including:
- `Header.tsx`: Main site header, navigation, and theme switcher. Uses `headerNavLinks` from config.
- `Footer.tsx`: Main site footer, renders columns and social links using `footerLinks` and `siteConfig`.
- `MobileNav.tsx`, `ScrollTop.tsx`, `ThemeSwitch.tsx`, `Analytics.tsx`, `PageTitle.tsx`, `SectionContainer.tsx`, `VideoPlayer.tsx`, `ActiveLink.tsx`, `Link.tsx`, `FooterSupportButton.tsx`, `Image.tsx`: Utility and layout components for navigation, theming, analytics, and content display.
- `SearchProvider.tsx`: Context provider for search functionality.
- `useThemeSwitch.tsx`: Custom hook for theme toggling.
- **[ui/](mdc:components/shared/ui)**: Shadcn-based UI primitives (buttons, toggles, dialogs, forms, etc.) for consistent design system usage. Example primitives:
  - `button.tsx`: Button component with multiple variants and sizes.
  - `toggle.tsx`: Toggle switch using Radix UI and class-variance-authority.
  - `dropdown-menu.tsx`, `menubar.tsx`, `pagination.tsx`, `calendar.tsx`, `card.tsx`, `table.tsx`, `tabs.tsx`, `select.tsx`, `sheet.tsx`, `slider.tsx`, `switch.tsx`, `badge.tsx`, `dialog.tsx`, `alert.tsx`, `avatar.tsx`, `form.tsx`, etc.
  - All primitives are designed for accessibility, theme support, and composability.

### [data/config/](mdc:data/config)

**Configuration Directory**: Centralized site and UI configuration:
- `site.settings.js`: Main site config (title, description, analytics, search, etc.), imported as `siteConfig`.
- `siteSettingsInterface.ts`: TypeScript interface for site config and metadata.
- `metadata.js`: Site metadata (title, description, domain, social links, theme, etc.).
- `footerLinks.ts`: Footer navigation columns and links.
- `headerNavLinks.ts`: Header navigation links, referencing `siteConfig` for dynamic paths.
- `colors.js`: Semantic color palette for Tailwind and UI.
- `pricingData.tsx` & `pricingDataInterface.ts`: Pricing tiers, frequencies, and types for pricing pages.
- `searchLinks.ts`: Quick navigation/search links for the app.

These config files are imported throughout the app and components to ensure consistent navigation, theming, and business logic.

## Other Notable Folders

- `.cursor/`: Cursor AI rules and metadata
- `public/`: Static assets

Refer to this rule for a high-level overview of the project structure and the purpose of each directory.

---
> Source: [PageAI-Pro/vibe-coding-starter](https://github.com/PageAI-Pro/vibe-coding-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
