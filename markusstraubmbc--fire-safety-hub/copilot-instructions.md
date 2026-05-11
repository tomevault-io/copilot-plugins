## fire-safety-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RESQIO Fire Safety Hub is a marketing/landing page website for a comprehensive fire department management system. Built with Vite, React, TypeScript, and shadcn/ui components, this site showcases various modules and features of the RESQIO platform for German-speaking fire departments (Feuerwehr).
## Development Commands

```bash
# Install dependencies (uses npm, but project supports bun as well)
npm i

# Start development server (Vite dev server on port 8080)
npm run dev

# Build for production
npm run build

# Build in development mode
npm run build:dev

# Lint code
npm run lint

# Preview production build
npm run preview
```

## Architecture Overview

### Tech Stack
- **Build Tool**: Vite 5.x with React SWC plugin for fast compilation
- **Framework**: React 18.3 with TypeScript 5.8
- **Routing**: React Router DOM v6 (client-side routing)
- **UI Framework**: shadcn/ui (Radix UI primitives + Tailwind CSS)
- **Styling**: Tailwind CSS with custom HSL-based theming
- **State Management**: TanStack Query for server state
- **Forms**: React Hook Form with Zod validation

### Project Structure

```
src/
├── components/          # Reusable components
│   ├── ui/             # shadcn/ui components (auto-generated, don't manually edit)
│   ├── Header.tsx      # Main navigation with scroll detection
│   ├── Footer.tsx      # Site footer
│   └── *Section.tsx    # Landing page sections (Hero, Features, Pricing, etc.)
├── pages/              # Route pages
│   ├── Index.tsx       # Main landing page (composes all sections)
│   ├── ModulDetail.tsx # Dynamic module detail pages
│   ├── Impressum.tsx   # Legal imprint
│   ├── Datenschutz.tsx # Privacy policy
│   └── NotFound.tsx    # 404 page
├── data/
│   └── module-data.ts  # Central module definitions (all features/modules)
├── hooks/              # Custom React hooks
├── lib/
│   └── utils.ts        # Utility functions (cn for className merging)
└── App.tsx             # Root component with providers and routing
```

### Key Design Patterns

1. **Module Data Architecture**: All RESQIO modules are defined in `src/data/module-data.ts`. This single source of truth contains:
   - Module metadata (title, descriptions, keywords)
   - Benefits and features lists
   - Technical details
   - Icon and color associations
   - SEO metadata

2. **Route Structure**:
   - `/` - Main landing page (Index.tsx)
   - `/modul/:slug` - Dynamic module detail pages using module-data.ts keys
   - `/impressum` - Legal imprint
   - `/datenschutz` - Privacy policy
   - `*` - Catch-all 404 route

3. **Component Composition**: The Index page is composed of discrete section components (HeroSection, FeaturesSection, etc.) arranged sequentially. Each section is self-contained with its own styling and data.

4. **Scroll-based Navigation**: Header.tsx implements smooth scrolling to section IDs on the homepage using the `scrollToSection` function. Navigation links scroll to anchored sections rather than navigate to new routes.

5. **Dynamic Meta Tags**: ModulDetail.tsx updates document title, description, and keywords dynamically based on the module being viewed.

## Styling System

### Tailwind Configuration
- **Base Color**: Slate
- **Theme System**: HSL-based CSS variables (see `src/index.css`)
- **Custom Fonts**:
  - Sans: Poppins
  - Serif: Merriweather
  - Mono: JetBrains Mono
- **Custom Shadows**: Defined via CSS variables (--shadow-xs through --shadow-2xl)

### Color Palette
The site uses a custom color system defined in tailwind.config.ts with semantic naming:
- `background`, `foreground` - Base colors
- `primary`, `secondary` - Brand colors
- `muted`, `accent` - Supporting colors
- `card`, `popover` - Component backgrounds
- Each module in module-data.ts can have its own color (blue, amber, red, slate, etc.)

### Path Alias
- `@/` maps to `./src/` (configured in vite.config.ts and tsconfig.json)

## Important Files

### module-data.ts
Central data source for all RESQIO modules. When adding or modifying module information:
- Use the ModuleData interface
- Include all required fields: title, shortDesc, longDesc, benefits, features, icon
- Optional: technicalDetails, keywords (for SEO), color
- Slug is the object key in the modules record

### Header.tsx
- Implements sticky header with scroll detection
- Background blur and styling changes on scroll
- Scroll progress bar at bottom
- Mobile responsive menu
- Uses `scrollToSection` for smooth scrolling to anchored sections

### App.tsx
- Sets up QueryClient for TanStack Query
- Wraps app in TooltipProvider, Toaster, and Sonner
- Defines all routes with BrowserRouter
- **CRITICAL**: All custom routes must be added ABOVE the catch-all `*` route

## shadcn/ui Components

This project uses shadcn/ui components located in `src/components/ui/`. These are:
- Auto-generated via the shadcn CLI
- Based on Radix UI primitives
- Styled with Tailwind CSS
- Should not be manually edited; use the CLI to update/add components

Configuration: `components.json`

## Content Language

All user-facing content is in German (Deutsch). Maintain German language for:
- UI text, buttons, navigation
- Marketing copy, descriptions
- Meta tags, page titles
- Error messages

## Common Development Patterns

### Adding a New Module
1. Add module entry to `modules` object in `src/data/module-data.ts`
2. Choose appropriate icon from lucide-react
3. Module automatically appears in listings and is accessible via `/modul/{key}`

### Adding a New Section to Landing Page
1. Create component in `src/components/` as `{Name}Section.tsx`
2. Import and add to `src/pages/Index.tsx`
3. Add anchor ID for scroll navigation

### Adding shadcn/ui Components
Use the shadcn CLI (already configured via components.json):
```bash
npx shadcn@latest add [component-name]
```

## TypeScript Configuration

- Relaxed type checking: `noImplicitAny: false`, `strictNullChecks: false`
- Allow JavaScript files: `allowJs: true`
- Path aliases configured for `@/*`
- Skip lib checks for faster builds

## Build Output

Production builds go to `dist/` directory. The project is configured for static site deployment.

## About RESQIO Platform

This website markets **RESQIO** - a comprehensive fire department management system (Feuerwehr-Verwaltungssoftware). RESQIO is the actual SaaS application, while this repository contains only the marketing/landing page.

### What RESQIO Does

RESQIO is an all-in-one platform for fire departments covering:
- **Equipment Management** (Ausrüstungsverwaltung): Inventory, maintenance, lifecycle tracking
- **Operations** (Einsatzmanagement): Mission documentation, exercises, reports
- **Personnel** (Personalverwaltung): Qualifications, training, availability analysis
- **Logistics**: Vehicle fleet, laundry, warehouse movements
- **Kiosk Mode**: Touchscreen-optimized interface for fire stations
- **AI Features**: Text optimization, personnel analysis, smart parsing
- **Enterprise**: MQTT integration, object plans (DIN 14095), digital ID cards

### Target Audience

German-speaking fire departments (Freiwillige Feuerwehr) with focus on:
- Municipalities < 5,000 inhabitants (All-in-One package)
- Municipalities < 10,000 inhabitants (Professional package)
- Cities and districts (Enterprise package)

### Pricing Tiers

| Package | Target | Annual Price |
|---------|--------|-------------|
| All in One | < 5,000 inhabitants | 399 € |
| Professional | < 10,000 inhabitants | 599 € |
| Enterprise | Cities/Districts | On request |

## Content Strategy & Documentation Sources

### Reference Documentation

The repository includes comprehensive German documentation about RESQIO features:

1. **WEBSITE_CONTENT.md** - Marketing copy for all modules (primary source for website content)
2. **FEATURES_DOKUMENTATION.md** - Detailed feature specifications (~1400 lines)
3. **HANDBUCH_ADMINISTRATOR.md** - Administrator handbook with technical details
4. **HANDBUCH_WARTUNG.md** - Maintenance management guide
5. **HANDBUCH_KI_FEATURES.md** - AI features handbook

### Using Documentation for Content Updates

When updating marketing copy or adding features:

1. **Check WEBSITE_CONTENT.md first** - This contains curated marketing descriptions for each module
2. **Reference FEATURES_DOKUMENTATION.md** - For technical accuracy and comprehensive feature lists
3. **Module naming conventions**:
   - Dashboard: "Kommandozentrale"
   - Equipment: "Gerätehaus & Technik"
   - Maintenance: "Wartungsmanagement"
   - Kiosk: "Kiosk-Modus"
   - Operations: "Einsätze & Übungen"
   - And 20+ additional modules

### Content Principles

- **Language**: All content in German (formal "Sie" form for business communication)
- **Tone**: Professional but accessible, emphasizing safety and efficiency
- **Keywords**: Focus on fire department terminology (DGUV, FwDV, DIN 14095, Atemschutz, etc.)
- **Benefits over features**: Emphasize "Mehrwert" (value) for fire departments
- **Real-world scenarios**: Reference actual use cases (Prüffristen, Einsatzbereitschaft, Wartung)

### Module Data Synchronization

When adding/updating modules in `module-data.ts`:
- Cross-reference with WEBSITE_CONTENT.md for accurate descriptions
- Maintain consistency between `shortDesc`, `longDesc`, and `benefits`
- Use appropriate icons from lucide-react (existing pattern: Map, AlertTriangle, FileText, Wrench, Users, etc.)
- Choose semantic colors that match the module's purpose

## Important Marketing Context

### Key Differentiators

RESQIO stands out through:
1. **All-in-One**: Complete solution, no need for multiple tools
2. **German Market Focus**: Compliance with DGUV, FwDV, DIN 14095
3. **Modern Tech Stack**: React 19, Node.js, MariaDB, Docker-ready
4. **AI Integration**: OpenAI-powered text optimization and analysis
5. **Kiosk Mode**: Tablet-optimized for on-site usage at fire stations
6. **Server Location**: Data hosted in Germany (DSGVO-compliant)

### Call-to-Actions

Primary CTAs throughout the site:
- "Jetzt Demo anfordern" (Request demo)
- "Angebot anfragen" (Request quote)
- "Kontakt aufnehmen" (Get in touch)

Contact: support@resqio.de

### SEO Keywords

Important terms for fire department market:
- Feuerwehr Software, Gerätewart, Wartungsplaner
- Einsatzverwaltung, Inventar, Ausrüstung
- DGUV, FwDV, DIN 14095, Atemschutz (AGT)
- Digitaler Dienstausweis, Kiosk-Modus
- Objektpläne, Hydrantenkarte

---
> Source: [markusstraubmbc/fire-safety-hub](https://github.com/markusstraubmbc/fire-safety-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
