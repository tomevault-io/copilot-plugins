## daniel-ives

> This is a minimal personal portfolio site built with Astro, TypeScript, and Tailwind CSS. The site is designed to be fast, lightweight, and clean - prioritizing minimal bundle size and performance.

# Daniel Ives Personal Site - Cursor Rules

## Project Overview
This is a minimal personal portfolio site built with Astro, TypeScript, and Tailwind CSS. The site is designed to be fast, lightweight, and clean - prioritizing minimal bundle size and performance.

## Tech Stack
- **Framework**: Astro 5.x (no React integration)
- **Language**: TypeScript
- **Styling**: Tailwind CSS with minimal custom CSS
- **Typography**: Nunito font (weights 400, 700 only)
- **Build**: Astro build system with TypeScript checking

## File Structure
- `src/pages/` - Astro pages (routes)
  - `src/pages/index.astro` - Homepage
  - `src/pages/about/` - About page
  - `src/pages/projects/` - Projects listing and detail pages
  - `src/pages/blog/` - Blog posts (skeleton)
- `src/components/` - Astro components only
  - `src/components/MainHead.astro` - Head component
- `src/layouts/` - Astro layout components
  - `src/layouts/BaseLayout.astro` - Base layout with centered content
  - `src/layouts/ProjectLayout.astro` - Layout for project detail pages
- `src/content/` - Markdown content files
  - `src/content/projects/` - Project markdown files
- `src/styles/` - Global styles
  - `src/styles/globals.css` - Minimal CSS with system theme support
- `public/` - Static assets (images, favicon)

## Path Aliases
- Use `@/` prefix for imports from `src/`
  - `@/components` → `src/components`
  - `@/layouts` → `src/layouts`
  - `@/styles` → `src/styles`
  - `@/assets` → `src/assets`

## Design Philosophy

### Minimalism
- Keep bundle size as small as possible
- No unnecessary JavaScript or dependencies
- Pure Astro components - no React
- Simple, clean typography (only 2 font sizes: header and body)
- Minimal color palette (grays + system theme)

### Typography
- **Header**: `text-4xl font-bold` for page titles
- **Body**: Default size for all other text
- Font: Nunito (weights 400 and 700 only)
- Line height: `leading-7` for better readability

### Color Scheme
- Light mode: `bg-gray-50` background, black text
- Dark mode: `bg-neutral-900` background, white text (via system preference)
- Links: Underlined, no color accents
- System theme: Automatically respects `prefers-color-scheme`

## Component Patterns

### Astro Components (`.astro`)
- Use for ALL components (no React)
- Use frontmatter (`---`) for imports and TypeScript
- Use `class` attribute (not `className`) for Tailwind classes
- Example:
```astro
---
import BaseLayout from "@/layouts/BaseLayout.astro";
const { prop } = Astro.props;
---

<BaseLayout>
  <div class="flex items-center">
    <p>{prop}</p>
  </div>
</BaseLayout>
```

## Styling Guidelines

### Tailwind CSS
- Use Tailwind utility classes for styling
- No custom CSS variables or complex theming
- Use responsive prefixes (`md:`, `lg:`, etc.) for breakpoints
- Keep it simple - prefer standard Tailwind colors

### Dark Mode
- Uses system preference (`prefers-color-scheme: dark`)
- No manual toggle - automatic based on user's OS setting
- Tailwind config: `darkMode: "media"`

### Spacing Consistency
**CRITICAL**: All spacing must be uniform and consistent across all pages.

#### Heading Spacing
- **H1 headings**: Always use `text-4xl font-bold mb-6` for page titles
- **H2 headings**: Use `font-bold mb-4` for section headings
- Never deviate from these patterns

#### Navigation Spacing
- **All pages with navigation**: Use the nav slot with absolute positioning
- Standard nav pattern: `hidden md:block absolute top-0 md:top-8 left-6 right-6 pb-2 border-b border-gray-200 dark:border-gray-700`
- Always use `slot="nav"` and `transition:name="page-nav"` for consistency
- Navigation should NOT affect main content alignment

#### Content Spacing
- **Main content sections**: Use `mb-6` for primary spacing between major sections
- **Paragraphs**: Use `mb-4` for regular paragraph spacing, `mb-6` for larger gaps
- **Lists**: Use `mb-8` for spacing after lists, `space-y-2` for list items
- **Project descriptions**: Use `mb-6` to match main content spacing

#### BaseLayout Spacing
- BaseLayout provides: `max-w-2xl mx-auto px-6 pt-6 md:pt-24 pb-12`
- All pages inherit this - do NOT add extra padding that shifts content alignment
- Content should align consistently across all pages

#### Projects Page Specific
- Timeline elements are absolutely positioned and should NOT affect main content
- Years positioned at `-left-40`, vertical line at `-left-8`
- Main content should have NO left padding (`md:pl-8` removed) to maintain alignment
- Timeline extends to the left without shifting main content to the right

#### Spacing Checklist
When creating or editing pages, ensure:
- ✅ H1 uses `mb-6` (not `mb-2` or other values)
- ✅ Navigation uses the standard absolute positioning pattern
- ✅ Main content sections use `mb-6`
- ✅ No extra padding that shifts content alignment
- ✅ Content aligns with other pages (check visually)

## TypeScript
- Use TypeScript for all `.ts` and `.tsx` files (though we don't use TSX)
- Define interfaces for component props
- Use type imports: `import type { ... }`
- Use strict typing - avoid `any` when possible

## Astro-Specific Patterns

### Content Collections
- Markdown files in `src/content/projects/` for projects
- Use frontmatter for metadata (title, date, description, quickLinks)
- Date format: "Season Year" (e.g., "Fall 2025", "Spring 2023")
- Access via Astro's content collections API

### Layouts
- `BaseLayout`: Centered content (`max-w-2xl`), responsive padding
- `ProjectLayout`: For individual project pages with navigation

## Code Style

### Naming
- Components: PascalCase (`MyComponent.astro`)
- Files: Match component name
- Functions: camelCase
- Constants: UPPER_SNAKE_CASE or camelCase

### Formatting
- Use 2 spaces for indentation
- Use single quotes for strings in TypeScript
- Use double quotes for strings in Astro templates
- Trailing commas in objects/arrays

### Imports
- Group imports: external packages → internal components → types
- Use absolute imports with `@/` prefix
- Sort imports logically

## Performance
- Pure Astro components (no client-side JavaScript unless absolutely necessary)
- Static generation for all pages
- Lazy load images with `loading="lazy"`
- Minimal CSS (no complex animations or transitions)
- Small font loading (only 2 weights)

## Build & Development
- `npm run dev` - Start dev server (localhost:4321)
- `npm run build` - Build for production (includes type checking)
- `npm run preview` - Preview production build
- Type checking runs automatically on build via `@astrojs/check`

## Common Patterns

### Conditional Rendering
- Use ternary or logical operators in Astro templates
- Example: `{condition && <div>Content</div>}`

### Asset Imports
- Static images in `public/` folder (served as-is, not processed)
- Use regular `<img>` tags with `loading="lazy"` for performance

### Props
- Access via `Astro.props` in frontmatter
- Define TypeScript interfaces for props

## Page-Specific Notes

### Homepage (`src/pages/index.astro`)
- Simple intro paragraph
- Links to About and Blog
- Recent projects section (currently just Moon Lander)
- External links at bottom

### Projects Page (`src/pages/projects/index.astro`)
- Timeline layout grouped by year
- Years on left, vertical line, projects on right
- Years sorted descending (newest first)
- Navigation hidden on mobile

### Project Detail Pages (`src/layouts/ProjectLayout.astro`)
- Title, date, description
- Simple, clean layout
- Uses prose plugin for markdown content

### About Page (`src/pages/about/index.astro`)
- Bio content with image
- Image on right side (desktop)

### Blog Page (`src/pages/blog/index.astro`)
- Currently skeleton/placeholder
- Ready for future blog posts

## Notes
- Site is deployed on Cloudflare Pages
- Minimal dependencies (only Astro, Tailwind, TypeScript, typography plugin)
- No React, no UI libraries, no icons libraries
- System theme support (no manual toggle)
- Mobile-first responsive design
- Navigation hidden on mobile (users can swipe back)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/daives01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
