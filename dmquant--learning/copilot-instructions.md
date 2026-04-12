## learning

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a C++ Learning Hub - an interactive educational website built with Astro and React. The site provides comprehensive C++ programming tutorials from basic syntax to advanced competitive programming concepts, organized into 6 progressive levels. Features full internationalization support with English and Chinese languages.

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Check TypeScript types
npx astro check
```

The development server runs at `http://localhost:4321`

## Architecture

### Technology Stack
- **Framework**: Astro 5.8+ with React integration
- **Language**: TypeScript with strict configuration
- **Styling**: CSS-in-JS with CSS custom properties for theming
- **Content**: Static site generation with `.astro` pages
- **Internationalization**: Built-in i18n support for English and Chinese
- **Syntax Highlighting**: Prism.js with tomorrow theme

### Project Structure
```
src/
├── layouts/Layout.astro          # Main layout with navigation & TOC
├── components/
│   ├── CodeExample.tsx          # Interactive code component
│   └── LanguagePicker.astro     # Language switcher component
├── i18n/
│   ├── ui.ts                    # Translation strings
│   └── utils.ts                 # i18n utility functions
└── pages/                       # Learning content pages
    ├── index.astro              # Language detection redirect
    ├── [lang]/index.astro       # Localized homepage
    ├── 1-syntax-basics/         # Level 1: C++ fundamentals
    ├── 2-algorithm-basics/      # Level 2: Core algorithms
    ├── 3-data-structures/       # Level 3: Data structures
    ├── 4-competitive/           # Level 4: Competitive programming
    ├── 5-advanced/              # Level 5: Advanced algorithms
    ├── 6-world-class/           # Level 6: World-class techniques
    └── [lang]/                  # Localized versions of all content
```

### Key Components

**Layout.astro** (`src/layouts/Layout.astro`):
- Global layout with responsive design
- Sidebar navigation with 6-level hierarchy
- Top navigation bar with breadcrumbs and language picker
- Table of contents auto-generation from headings
- Dark/light theme toggle with localStorage persistence
- Reading progress tracking with animated progress bar
- Mobile-responsive with collapsible sidebars

**CodeExample.tsx** (`src/components/CodeExample.tsx`):
- Interactive code blocks with syntax highlighting (Prism.js)
- Copy-to-clipboard functionality
- Toggle between code and output views
- Supports explanations and expected output
- Requires `client:load` directive for hydration

**LanguagePicker.astro** (`src/components/LanguagePicker.astro`):
- Dropdown language switcher
- Maintains current route when switching languages
- Styled with CSS custom properties for theme consistency

### Internationalization (i18n)

The site supports two languages configured in `astro.config.mjs`:
- **English** (`en`) - Default language
- **Chinese** (`zh`) - Traditional Chinese

**URL Structure**:
- Root `/` redirects to browser language or fallback to `/en/`
- Localized routes: `/en/1-syntax-basics/variables` and `/zh/1-syntax-basics/variables`
- Language switching preserves current route

**Translation System**:
- All strings centralized in `src/i18n/ui.ts`
- Helper functions in `src/i18n/utils.ts` for language detection
- Navigation and content fully translated

### Content Organization

Navigation hierarchy (actual folder structure):
- **Level 1**: Syntax Basics (7 lessons) - Variables, functions, classes, pointers
- **Level 2**: Algorithm Basics (7 lessons) - Sorting, searching, recursion, DP
- **Level 3**: Data Structures (9 lessons) - Arrays, vectors, trees, graphs, hash tables
- **Level 4**: Competitive Programming (16 lessons) - STL, segment trees, advanced algorithms
- **Level 5**: Advanced Algorithms (4 lessons) - Graph theory, Union-Find
- **Level 6**: World-Class Level (11 lessons) - Combinatorics, advanced DP, balanced trees

Each level has:
- **Index page**: Overview with lesson cards
- **Individual lessons**: Detailed content with code examples

### Styling System

**CSS Architecture**:
- CSS custom properties for consistent theming
- Light/dark mode via `data-theme` attribute on `<html>`
- Theme state persisted in localStorage
- Responsive design with mobile-first approach

**Layout System**:
- Desktop: 3-column layout (sidebar, content, TOC)
- Mobile: Collapsible sidebars with overlay
- Sticky headers and navigation elements

## Content Development

### Adding New Lessons
1. Create `.astro` file in appropriate level directory (`1-syntax-basics/`, etc.)
2. Use `Layout.astro` for consistent structure
3. Import and use `CodeExample.tsx` for interactive code
4. Update sidebar navigation in `Layout.astro` if adding new pages
5. Add translations to `src/i18n/ui.ts` for both languages
6. Create localized versions in `[lang]/` directories

### CodeExample Usage
```astro
---
import Layout from '../../layouts/Layout.astro';
import CodeExample from '../../components/CodeExample.tsx';
---

<Layout title="Lesson Title">
  <CodeExample 
    client:load
    title="Example Title"
    code={`#include <iostream>
using namespace std;

int main() {
    cout << "Hello World!" << endl;
    return 0;
}`}
    explanation="Brief explanation of what this code demonstrates"
    output="Hello World!"
  />
</Layout>
```

### Adding New Languages
1. Add language code to `astro.config.mjs` locales array
2. Add language entry to `src/i18n/ui.ts` languages object
3. Create translation strings in `ui.ts` for the new language
4. Create localized page directories in `src/pages/[lang]/`

## Development Notes

- All pages use the shared `Layout.astro` for consistent navigation and styling
- TypeScript configured with strict mode via `astro/tsconfigs/strict`
- Prism.js provides syntax highlighting with `prism-tomorrow` theme
- Interactive React components require `client:load` directive
- Site is statically generated - no server-side rendering
- CSS variables enable consistent theming across components
- Mobile responsiveness handled through CSS media queries in Layout.astro
- Reading progress and TOC generation handled via vanilla JavaScript in Layout.astro

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dmquant)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dmquant)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
