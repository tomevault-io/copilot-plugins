## my-fullstack-dev-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Next.js 16 personal portfolio website** built with **TypeScript** and **Tailwind CSS**. It's a modern, responsive single-page application showcasing professional work, skills, and experience with dark mode support and scroll-triggered animations.

## Technology Stack

- **Framework:** Next.js 16.0.1 (App Router)
- **Language:** TypeScript 5
- **Styling:** Tailwind CSS 3.3.0 + PostCSS
- **UI Libraries:** React 18, Lucide React, FontAwesome
- **Animations:** AOS (Animate on Scroll)
- **Theme Management:** next-themes
- **Linting:** ESLint (Airbnb preset) + Prettier
- **Deployment:** Vercel (configured via .vercel/)

## Common Development Commands

```bash
# Start development server (localhost:3000)
npm run dev

# Build for production
npm run build

# Run production build locally
npm start

# Run ESLint checks
npm run lint

# Auto-fix linting and formatting issues
npm run fix
```

## Directory Structure

```
src/app/
├── components/           # Feature components (Header, Skills, Projects, Timeline, Footer)
├── integration/          # Integration page with Allaw legal directory embed
├── layout.tsx           # Root layout wrapper
├── page.tsx             # Home/index page
├── Providers.tsx        # Theme provider configuration
├── ThemeSwitcher.tsx    # Dark/light mode toggle component
└── globals.css          # Global styles
```

**Key Pattern:** File-based routing using Next.js App Router. Nested pages are defined by directory structure (e.g., `integration/page.tsx` → `/integration` route).

## Architecture Overview

### 1. **App Router & Page Structure**

- Uses modern Next.js 16 App Router, not Pages Router
- Root layout in `layout.tsx` wraps all pages
- Home page in `page.tsx`
- Nested pages in subdirectories (e.g., `integration/page.tsx`)

### 2. **Component Architecture**

- **Page Components:** Full-page sections (Header, Skills, Projects, Timeline, Footer)
- **Utility Components:** MenuButton (theme toggle), ThemeSwitcher (theme provider)
- Heavy use of `"use client"` directive for client-side interactivity
- Components use React hooks (useState, useEffect, useRef) for state management

### 3. **Theme Management**

```typescript
// Providers.tsx implements theme provider with hydration safety
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
// Returns null until mounted to prevent hydration mismatch
```

Theme switching uses `next-themes` with class-based dark mode. Apply dark styles with `dark:` Tailwind prefix.

### 4. **Styling Approach**

- **Utility-first CSS:** All styling via Tailwind classes
- **Responsive Breakpoints:** Custom in `tailwind.config.ts` (xs: 320px, 2xs: 390px, 3xs: 480px)
- **Dark Mode:** Class-based, configured in Tailwind config
- **Global Styles:** `globals.css` for base styles

### 5. **Animation Pattern**

- AOS library for scroll-triggered animations
- Initialized in component useEffect: `AOS.init({ duration: 200 })`
- Applied via `data-aos` attributes on elements

### 6. **Event Handling Conventions**

- Click-outside detection using refs and event listeners
- Always clean up event listeners in useEffect return
- Use `useRef` for DOM element references

## Configuration Files

- **tsconfig.json:** Strict mode enabled, path alias `@/*` → `./src/*`
- **tailwind.config.ts:** Custom breakpoints, dark mode class-based
- **next.config.js:** Minimal configuration
- **.eslintrc.json:** Airbnb preset, disallows console except warn/error/info
- **.prettierrc:** Default Prettier configuration

## Code Style & Conventions

- **TypeScript Strict Mode:** Required for all new files
- **File Naming:** PascalCase for components (`Header.tsx`), kebab-case for directories (`integration/`)
- **Variables & Functions:** camelCase
- **HTML/CSS IDs & Classes:** Descriptive, semantic
- **Component Props:** Define interfaces for all props
- **Type Safety:** Avoid `any` type; use proper TypeScript interfaces

## Important Notes

### ESLint Configuration

- Extends Airbnb preset with Next.js rules
- Disallows `console` statements except `warn`, `error`, `info`
- Auto-fix with `npm run fix`

### Client-Side Rendering

- This is a primarily client-side portfolio. Most pages use `"use client"` directive
- Minimal server-side logic; no API routes currently configured

### No Testing Framework

- Project currently lacks unit/integration tests
- Consider adding Jest/Vitest if expanding functionality

### Responsive Design

- Mobile-first approach with Tailwind
- Custom breakpoints optimized for various screen sizes
- Test changes on mobile, tablet, and desktop viewports

### External Integrations

- **Integration Page:** Uses Allaw embed script (dynamically loaded with cleanup)
- **Social Links:** GitHub, LinkedIn, Twitter, Malt
- **CV:** PDF file served from public/ directory

## Adding New Features

**New Component:**

1. Create file in `src/app/components/YourComponent.tsx`
2. Add `"use client"` at the top if using hooks
3. Use TypeScript interfaces for props
4. Import and use in appropriate page or parent component

**New Page:**

1. Create directory `src/app/your-page/`
2. Create `page.tsx` inside (file-based routing)
3. Import layout and components as needed

**Styling:**

- Use Tailwind utility classes
- Add dark mode support with `dark:` prefix
- Avoid custom CSS unless absolutely necessary (use globals.css only for base styles)

## Deployment

The repository is configured for **Vercel deployment** (`.vercel/` in .gitignore). Deploy directly from the main branch or use Vercel's GitHub integration for automatic deployments on push.

**Build Output:** Next.js generates `.next/` directory (in .gitignore)

## Internationalization (i18n) Implementation

### Overview

The portfolio has been enhanced with multi-language support using **next-intl** library. Users can switch between English and French, with automatic locale detection based on their browser preferences.

### Technology

- **Library:** next-intl v4.5.3
- **Locales:** English (en), French (fr)
- **Default:** English
- **Detection Strategy:** URL path > Cookie/localStorage > Accept-Language header > English

### File Structure

```
├── middleware.ts                # Locale detection and routing
├── i18n.config.ts             # Locale configuration
├── i18n.ts                      # next-intl configuration
├── src/i18n/
│   ├── en.json                 # English translations
│   └── fr.json                 # French translations
├── src/app/
│   ├── LanguageSwitcher.tsx    # Language toggle component
│   ├── [locale]/               # Dynamic locale route group
│   │   ├── layout.tsx          # Locale-aware root layout
│   │   ├── page.tsx            # Home page
│   │   ├── integration/
│   │   │   └── page.tsx        # Integration page
│   │   └── components/         # Shared components
│   └── (other files...)
```

### Key Features

**1. Locale Routing**

- All routes are prefixed with locale: `/en/`, `/fr/`
- Middleware automatically redirects unprefixed routes
- Browser language preference is respected for first visit
- User preference is stored in cookie `NEXT_LOCALE` for persistence

**2. Dynamic Metadata**

- Page title and description update based on locale
- Uses `generateMetadata()` in layout

**3. Translation Integration**

- Components use `useTranslations()` hook from next-intl
- Translations organized by feature: header, footer, skills, projects, timeline, common, metadata
- Supports nested translation keys: `t('header.greeting')`

**4. Language Switcher**

- Menu component with EN/FR toggle buttons
- Integrated into main menu alongside dark/light mode toggle
- Updates URL and stores preference in localStorage

### Adding Translations

**To add a new translated string:**

1. Add key-value pair to `src/i18n/en.json` and `src/i18n/fr.json`

```json
{
  "section": {
    "key": "English text here"
  }
}
```

2. Use in component with `useTranslations()`:

```typescript
import { useTranslations } from "next-intl";

function MyComponent() {
  const t = useTranslations("section");
  return <div>{t("key")}</div>;
}
```

### Supported Languages

| Language | Locale | File               | Status      |
| -------- | ------ | ------------------ | ----------- |
| English  | en     | `src/i18n/en.json` | ✅ Complete |
| Français | fr     | `src/i18n/fr.json` | ✅ Complete |

**To add a new language:**

1. Add locale to `i18n.config.ts` locales array
2. Create new translation file: `src/i18n/[locale].json`
3. Copy English keys and translate values
4. Update LanguageSwitcher component UI

### Configuration Files

**middleware.ts:**

- Intercepts all requests
- Detects user locale preference
- Redirects unprefixed routes to localized versions
- Fallback chain: URL → Cookie → Accept-Language → Default (en)

**i18n.config.ts:**

- Defines supported locales
- Sets default locale
- Exports Locale type for type safety

**i18n.ts:**

- Configures next-intl request handler
- Loads appropriate translation JSON files
- Validates locale against supported locales

### Development Notes

- Build process validates all translation keys
- Missing translations will cause build errors
- ESLint enforced for proper formatting
- TypeScript strict mode enabled for type safety

### Troubleshooting

**"Couldn't find next-intl config file" error:**

- Ensure `i18n.ts` exists in project root
- Check that middleware.ts is properly configured
- Verify route structure uses `[locale]` parameter

**Language switch not working:**

- Check browser accepts cookies (privacy settings)
- Verify URL includes locale prefix (e.g., `/en/`, `/fr/`)
- Clear browser cache and localStorage

**Missing translations:**

- Compare keys in `en.json` and `fr.json`
- Ensure JSON syntax is valid
- Check component uses correct `useTranslations()` namespace

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vincedsb1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
