## nuxt4-holistic-therapy-clinic

> - **Framework:** Nuxt 4.x (Compatibility Date: 2025-10-08).

# Nuxt 4 Monorepo Architectural Rules

## 1. Project Overview
- **Framework:** Nuxt 4.x (Compatibility Date: 2025-10-08).
- **Package Manager:** pnpm v10.x with Workspaces.
- **Core Architecture:** Multi-layered Monorepo using the `extends` feature.

## 2. Directory Structure & Layers
- **Root App:** Located in `/app`. Contains the main application logic, pages, and layouts.
- **Layers Directory:** `/layers`.
  - `@monorepo/theme-manager`: Found in `/layers/theme-manager`. Handles global CSS (`main.css`), Tailwind configuration, and Vite styling optimizations.
  - `@monorepo/ui-library`: Found in `/layers/ui-library`. Extends `theme-manager` and houses shared UI components.
- **Server Engine:** Nitro using the `server/` directory for API routes and sitemaps.

## 3. Coding Standards
- **Nuxt 4 Folders:** Follow the Nuxt 4 "Holistic" structure. Place Vue files in `app/`, server logic in `server/`, and shared utils in `shared/`.
- **Styling:** Use Tailwind CSS v4 via `@tailwindcss/vite`.
- **State Management:** Use Pinia via `@pinia/nuxt`.
- **Content:** Use Nuxt Content v3 with the SQLite database driver.

## 4. Layer Extension Rules
- All new UI features should be evaluated for placement in `@monorepo/ui-library` before being added to the local `app/` directory.
- The root `nuxt.config.ts` must extend `@monorepo/ui-library`, which in turn extends `@monorepo/theme-manager`.

## 5. Automation Commands
- **Scaffold Layer:** Create a new folder in `/layers`, add `nuxt.config.ts`, `package.json`, and `tsconfig.json`.
- **Verify Build:** Run `pnpm build` to ensure layer merging is successful.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/steve-r-lewis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/steve-r-lewis)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
