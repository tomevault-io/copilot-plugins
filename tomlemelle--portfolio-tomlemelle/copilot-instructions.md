## portfolio-tomlemelle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multilingual portfolio website for Tom Lemelle (Web Developer & Photographer) built with React, TypeScript, Vite, and Tailwind CSS. The site showcases web development projects, photography work, and videography services with support for English, French, and Italian.

## Build Commands

```bash
# Development server
npm run dev

# Production build
npm run build

# Lint code
npm run lint

# Preview production build
npm run preview
```

## Architecture

### Routing Structure
The application uses React Router with a layout-based structure:
- **Layout Component**: [Layout.tsx](src/components/Layout.tsx) wraps all routes with Navbar, Footer, and LanguageSwitcher
- **Routes**:
  - `/` - Home page
  - `/projects` - Projects listing page
  - `/projects/:id` - Individual project detail pages

### Internationalization System
Custom translation system without third-party i18n libraries:
- **Translation Provider**: [TranslationProvider.tsx](src/contexts/TranslationProvider.tsx) - Context-based system supporting `fr`, `en`, `it`
- **Locale Storage**: Uses `localStorage` to persist language preference
- **Translation Files**:
  - Main translations: `src/locales/{lang}.json`
  - Projects: `src/locales/projects/{lang}.json`
  - Testimonials: `src/locales/testimonials/{lang}.json`
- **Usage**: Import `useTranslation()` hook to access `locale`, `dictionary`, and `changeLanguage()`
- **Default Language**: French (`fr`)

### Project Data Architecture
Projects are **not** fetched from an API or database. They are:
- Stored as JSON files per language in `src/locales/projects/`
- Each project has a detailed `content` field containing markdown text
- Loaded dynamically based on current locale in [ProjectDetailPage.tsx](src/pages/ProjectDetailPage.tsx:17-21)
- Images stored in `public/images/realisations/`

### Styling System
- **Tailwind CSS** with custom configuration
- **CSS Variables**: Primary and secondary colors defined in [index.css](src/index.css:7-13) as RGB values
  - `--primary: 59, 144, 120` (teal green)
  - `--secondary: 59, 73, 144` (blue)
- **Custom Classes**: Glassmorphism effects (`.glass`, `.glass-dark`, `.glass-card`)
- **Background**: Grid pattern using CSS gradients
- **Font**: Poppins from Google Fonts

### Animation Libraries
- **Framer Motion**: Used for page transitions and loading animations
- **Swiper**: For image carousels in project detail pages with Navigation and Pagination modules

### Key Components
- **Navbar**: Fixed navigation with scroll-based styling changes
- **LanguageSwitcher**: Fixed position language selector (flags: FR, EN, IT)
- **Layout**: Includes a session-based loading animation (gear icon) shown only on first visit
- **ProjectDetailPage**: Uses `ReactMarkdown` with `remark-gfm` plugin to render project content

### Type Definitions
All TypeScript interfaces defined in [src/types/index.ts](src/types/index.ts):
- `Project` - Includes `categories`, `technologies`, `images[]`, `content` (markdown)
- `Service` - Service offerings
- `Testimonial` - Client testimonials
- `Language` - Supported locales

## Development Notes

### Adding New Projects
1. Add project data to all three locale files: `src/locales/projects/{en,fr,it}.json`
2. Ensure project `id` is unique and consistent across all languages
3. Add images to `public/images/realisations/`
4. Use relative paths (without leading `/public`) for `thumbnail` and `images[]`
5. Write project content in markdown format in the `content` field

### Adding New Translations
1. Add keys to main translation files: `src/locales/{en,fr,it}.json`
2. Access via `const { dictionary } = useTranslation()` then `dictionary.yourKey`
3. For nested objects, use dot notation: `dictionary.home.hero.heading`

### Styling Conventions
- Use Tailwind utility classes primarily
- Custom glassmorphism effects via `.glass`, `.glass-card` classes
- Colors: Use `bg-primary`, `text-primary`, etc. for theme colors
- Animations: Custom Tailwind animations defined for floating effects (`float`, `float-delay`, `float-delay-2`)

### Image Handling
- Static images in `public/images/` directory
- Reference with leading `/` in src attributes: `src="/images/..."`
- In ProjectDetailPage, images are prefixed with `/` when loaded: `src={"/" + image}`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TomLemelle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
