## portfoliowebsite-2026

> this file provides guidance to gemini when working with code in this repository.

# GEMINI.md

this file provides guidance to gemini when working with code in this repository.

## project overview

this is a modern, minimalist portfolio template built with astro and tailwind css v4. it's designed to be easily customizable through a single configuration file while maintaining a clean, professional appearance.

## tech stack

- **astro**: static site generator
- **tailwind css v4**: utility-first css framework using the new @tailwindcss/vite plugin
- **typescript**: for type-safe configuration
- **tabler icons**: icon library

## development commands

```bash
npm run dev       # start development server
npm run build     # build for production
npm run preview   # preview production build
```

## architecture

the project follows a component-based architecture with all customization centralized in `src/config.ts`:

- **components** (`src/components/`): individual astro components for each section (hero, about, projects, experience, education, header, footer)
- **main layout** (`src/pages/index.astro`): single-page layout that imports all components
- **configuration** (`src/config.ts`): single source of truth for all content and customization

### key architectural decisions

1. **single configuration file**: all content is managed through `src/config.ts` to make customization simple
2. **conditional rendering**: sections automatically hide if their data is removed from the config
3. **component independence**: each section is a self-contained component that reads from the config
4. **accent color system**: single `accentcolor` in config propagates throughout the site via css custom properties

## important implementation details

- the site uses tailwind css v4 with the vite plugin configuration
- no linting or testing framework is currently configured
- all components are in `.astro` format (not react/vue/etc)
- the project uses ibm plex mono font loaded from google fonts
- social links in the config are all optional and will conditionally render

## working with components

when modifying components:
1. components read directly from the imported `siteconfig` object
2. use tailwind utility classes for styling
3. maintain the existing monospace font aesthetic
4. use tabler icons for consistency with existing icons

## configuration structure

the `src/config.ts` exports a `siteconfig` object with these sections:
- basic info: name, title, description, accentcolor
- social links: email, linkedin, twitter, github (all optional)
- aboutme: string
- skills: string[]
- projects: array of {name, description, link, skills}
- experience: array of {company, title, daterange, bullets}
- education: array of {school, degree, dateRange, achievements}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KevinGastelum)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KevinGastelum)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
