## matiasvia-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server at localhost:4321
npm run build     # Build to ./dist/
npm run preview   # Preview production build locally
npm run astro     # Run Astro CLI commands (e.g. astro check for type errors)
```

There is no test runner or linter configured.

## Architecture

Single-page portfolio site built with **Astro 5** and **Tailwind CSS v4**. All sections live as Astro components assembled in `src/pages/index.astro` in this order: Navigation → Hero → About → Skills → Certifications → Projects → Contact → Footer.

**Tailwind v4 setup**: Uses `@tailwindcss/vite` plugin (not PostCSS). Component `<style>` blocks must include `@reference "../styles/global.css";` at the top to access Tailwind utilities via `@apply`.

**Dark mode**: Defaults to dark. Preference is stored in `localStorage.theme` and the `dark` class is toggled on `<html>`. An inline script in `Layout.astro` applies the class before paint to prevent flash.

**Icons**: All skill/technology icons are PNG files centralized in `src/assets/iconos/index.ts` and exported via the `ICONOS` const object.

**Contact form**: Uses EmailJS (`@emailjs/browser`). The Service ID, Template ID, and Public Key are hardcoded directly in `src/components/Contact.astro`. See `CONFIGURAR_EMAILJS.md` for setup instructions.

**Project detail pages**: Located under `src/pages/projects/`. Currently only `autoclinic.astro` exists as a stub.

**`lucide-react` is listed as a dependency but is not used** — no React integration is configured.

---
> Source: [mativia/matiasvia-web](https://github.com/mativia/matiasvia-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
