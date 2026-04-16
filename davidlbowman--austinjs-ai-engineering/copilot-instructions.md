## austinjs-ai-engineering

> <!-- markdownlint-disable-file MD013 -->

<!-- markdownlint-disable-file MD013 -->
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-page static site about agentic coding, teaching people how to use multiple agents simultaneously with Claude Code and GitHub CLI. Built with Astro v5.11.0, React, Tailwind CSS v4, and shadcn/ui components.

## Common Commands

### Development

```bash
bun dev          # Start dev server at localhost:4321
bun build        # Build for production to ./dist/
bun preview      # Preview production build locally
```

### Code Quality

```bash
bun format       # Format code using Biome
bun lint         # Lint code using Biome  
bun check        # Run both format and lint checks
```

### Testing

```bash
bun typecheck    # Run TypeScript type checking
bun test:e2e     # Run Playwright E2E tests with text output
```

### Dependency Management

```bash
bun install      # Install dependencies
bun add <pkg>    # Add new dependency
```

## Architecture

### Technology Stack

- **Framework**: Astro with React integration
- **Styling**: Tailwind CSS v4 (using new Vite plugin approach)
- **Components**: shadcn/ui with Radix UI primitives
- **Language**: TypeScript with strict mode
- **Package Manager**: Bun
- **Code Quality**: Biome for formatting and linting
- **Testing**: Playwright for E2E tests (multi-browser: Chrome, Firefox, Edge, Mobile)

### Project Structure

- `src/pages/` - Astro pages (file-based routing)
- `src/components/ui/` - shadcn/ui components
- `src/lib/` - Utility functions
  - `utils.ts` - Includes `cn()` for className merging
- `src/styles/global.css` - Global styles with Tailwind directives and CSS variables
- `e2e/` - Playwright E2E tests (files must end with `-e2e.ts`)
- `public/` - Static assets served directly

### Key Configurations

- **TypeScript**: Path alias `@/*` maps to `./src/*`
- **Tailwind CSS**: Configured via astro.config.mjs using Vite plugin
- **shadcn/ui**: Configured in components.json with "new-york" style and CSS variables
- **Biome**: Auto-formatting on file modifications
- **Playwright**: Configured for textual output to enable test result reading
- **E2E Test Pattern**: Files in `e2e/` directory must end with `-e2e.ts`

## Frontend Development Patterns

### Core Philosophy

**React and Shadcn take priority in ALL cases**.

### Component Development

- Use shadcn/ui components directly - never modify them
- Add new components with: `bunx shadcn@latest add <component-name>`
- Use `cn()` utility for conditional classes:

  ```typescript
  import { cn } from "@/lib/utils"
  
  <div className={cn(
    "base-classes",
    condition && "conditional-classes"
  )} />
  ```

### CSS Architecture

- Tailwind CSS v4 with CSS custom properties for theming
- Dark mode support via CSS variables in src/styles/global.css
- Always use `cn()` for conditional classes
- Follow Tailwind's utility-first approach

## Testing Patterns

### E2E Testing with Playwright

**Configuration**: Tests output text format for Claude Code readability

**Test Structure**:

```typescript
import { test, expect } from '@playwright/test'

test.describe('Feature Name', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/')
  })

  test('should display expected content', async ({ page }) => {
    await expect(page.getByRole('heading', { name: 'Title' })).toBeVisible()
    await expect(page.getByRole('button', { name: 'Action' })).toBeVisible()
  })
})
```

**Best Practices**:

- Test user-visible behavior
- Use `getByRole()`, `getByText()` over CSS selectors
- Be specific with selectors when multiple elements exist
- Use `await expect()` for auto-retrying assertions

## MCP Tools Setup

The project is designed to demonstrate MCP (Model Context Protocol) tools integration. Future setup will include tools for enhanced development workflows.

## Development Workflow

### Quality Assurance Protocol

After completing EACH todo item, run these verification scripts:

```bash
bun typecheck    # Ensure TypeScript types are correct
bun check        # Run Biome format and lint checks
bun test:e2e     # Run Playwright E2E tests
```

**CRITICAL**: Never proceed to the next todo until all checks pass.

### Task Management

1. User assigns tasks
2. Create todos for each task
3. Complete one todo at a time
4. Run ALL verification scripts after each todo
5. Fix any issues before proceeding
6. Mark todo as completed only when all checks pass

## Important Notes

- **Static Site Focus**: This is a single-page site - avoid overcomplicating
- **Testing Output**: Playwright tests must use text reporter for readability
- **No Backend**: This is a static site with no server-side logic
- **Educational Purpose**: Code should be clear and demonstrate best practices for teaching
- **Quality Gates**: Every change must pass typecheck, biome check, and E2E tests

## UI/UX Strategy: Converting README to Interactive Single-Page App

### Design Philosophy

Transform the technical README content into an engaging, interactive experience using shadcn/ui components with creative layouts and micro-interactions. Each section should feel distinct while maintaining visual cohesion.

### Lessons Learned from Implementation

#### Component Architecture

- **Keep it simple**: No unnecessary state management, avoid useEffect when not needed
- **Use Astro patterns**: Components need `client:load` directive for interactivity
- **Import order**: Biome will auto-organize imports (external libs first, then internal)
- **Section-specific components**: Create one component per content section for clarity

#### Styling Approach

- **Global styles belong in global.css**: Typography (h1, h2), section layouts, utility classes
- **Use Tailwind defaults**: Prefer `container` class over custom max-widths
- **Accent colors sparingly**: Only on key phrases, defined as utility classes
- **Let shadcn handle defaults**: Don't override colors/fonts until final polish
- **Boring sections pattern**: For text-heavy sections (MCP, Agentic Coding, Conclusion), use simple container + max-width wrapper + paragraphs

#### Global CSS Patterns

```css
@theme inline {
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
}

@layer base {
  body {
    @apply bg-purple-50/50 dark:bg-purple-900/10 text-foreground font-sans;
  }
  h1 {
    @apply text-5xl font-bold tracking-tighter sm:text-6xl xl:text-7xl/none font-mono;
    @apply bg-gradient-to-r from-foreground via-purple-600 to-foreground bg-clip-text text-transparent;
  }
  h2 {
    @apply text-3xl font-bold tracking-tighter sm:text-4xl xl:text-5xl font-mono;
    @apply bg-gradient-to-r from-foreground via-purple-500 to-foreground bg-clip-text text-transparent;
  }
  h3 {
    @apply text-2xl font-bold sm:text-3xl xl:text-4xl font-mono;
    @apply bg-gradient-to-r from-foreground via-purple-500 to-foreground bg-clip-text text-transparent;
  }
  h4, h5, h6 {
    @apply font-mono font-semibold;
  }
  p {
    @apply text-lg text-foreground;
  }
  section {
    @apply py-12 px-4 sm:py-16;
  }
}

@layer utilities {
  .text-accent-purple {
    @apply text-purple-600 dark:text-purple-400 font-bold;
  }
  .drop-cap::first-letter {
    @apply float-left text-7xl md:text-8xl font-bold leading-[0.8] mr-2 mt-1;
    @apply bg-gradient-to-br from-purple-600 to-purple-400 bg-clip-text text-transparent;
  }
  .highlight {
    @apply relative inline;
    background-image: linear-gradient(
      104deg,
      transparent 0.4rem,
      oklch(0.85 0.15 302 / 0.35) 0.4rem,
      oklch(0.85 0.15 302 / 0.35) calc(100% - 0.2rem),
      transparent calc(100% - 0.2rem)
    );
    padding: 0.375rem 0.5rem 0.25rem 0.5rem;
    margin: -0.25rem -0.375rem -0.125rem -0.375rem;
    box-decoration-break: clone;
    -webkit-box-decoration-break: clone;
    filter: drop-shadow(0 0 0.5px oklch(0.85 0.15 302 / 0.2));
    @apply dark:bg-gradient-to-r dark:from-transparent dark:via-purple-500/20 dark:to-transparent;
  }
}
```

#### Section Design Patterns

1. **Interactive Sections** (Hero, Current State, Pros/Cons, AI Scalar, Context, Compute):
   - Use cards, badges, and interactive elements
   - Add visual interest with icons and animations
   - Create engaging layouts with grids and varied spacing

2. **Text-Heavy "Boring" Sections** (MCP, Agentic Coding, Context-Compute Relationship, Conclusion):

   ```tsx
   <section>
     <div className="container mx-auto">
       <h2 className="text-center mb-12">Section Title</h2>
       <div className="max-w-4xl mx-auto space-y-4">
         <p>Content paragraph...</p>
         <p>Another paragraph...</p>
       </div>
     </div>
   </section>
   ```

   - Simple container structure
   - Centered title
   - Left-aligned paragraphs with max-width constraint
   - No cards or decorative elements

3. **Mixed Sections** (Preparation):
   - Combine both approaches
   - Use cards for actionable items
   - Plain text for explanatory content

#### Testing Strategy

- **Keep E2E tests minimal**: Just verify sections exist, not specific content
- **Focus on functionality**: Test interactions when added, not static content
- **Use semantic queries**: `page.locator('section')` over specific classes

#### Development Workflow

- **Run all checks after changes**: `bun typecheck`, `bun check`, `bun test:e2e`
- **Let tools auto-fix**: Biome handles formatting, don't fight it
- **Use MCP tools when needed**: Astro MCP for config

#### Recent Changes (Session Summary)

**Previous Session**:
1. **Simplified text-heavy sections**: Converted MCP, Agentic Coding, and Conclusion sections to boring paragraph format
2. **Removed decorative elements**: Eliminated cards and icons from text-heavy sections for better readability
3. **Standardized layout**: All boring sections now use consistent container/max-width/paragraph structure
4. **Fixed Context-Compute Relationship**: Removed icon decoration to match boring section style

**Current Session - Purple Theme & Typography**:

1. **Updated H2 styling**: Added gradient effect similar to H1 using `via-purple-500`
2. **Implemented purple color scheme**: 
   - Near-black purple for main text
   - Off-white purple backgrounds using OKLCH color space
   - Updated all CSS variables in both light and dark modes
3. **Updated icon colors**: Changed all icons to use `text-purple-500 dark:text-purple-400`
4. **Background improvements**:
   - Added global purple-tinted background to body
   - Removed individual section backgrounds
   - Created consistent purple-tinted backgrounds for cards/highlights
5. **Font implementation using Astro's experimental fonts API**:
   - Configured Geist Sans and Geist Mono via `fontProviders.fontsource()`
   - Added Font components to page head with preload
   - Defined `--font-sans` and `--font-mono` in Tailwind theme
   - Body text uses Geist Sans, all headers use Geist Mono
6. **Typography improvements**:
   - H1: `text-5xl sm:text-6xl xl:text-7xl` with gradient
   - H2: `text-3xl sm:text-4xl xl:text-5xl` with gradient
   - H3: `text-2xl sm:text-3xl xl:text-4xl` with gradient
   - H4-H6: Geist Mono with semibold weight
   - All paragraphs: `text-lg` minimum
7. **Drop caps implementation**:
   - Added `.drop-cap::first-letter` utility class
   - Applied to first paragraphs in text-heavy sections
   - Purple gradient styling for drop caps

**Latest Session - Mobile Fixes & Highlights**:

1. **Mobile styling fixes**:
   - Fixed alignment issues on small screens
   - Adjusted padding for better mobile experience (`px-4` consistently)
   - Fixed Context section percentage inconsistencies
   - Improved responsive grid layouts (3-2 layout for context boxes on mobile)
   
2. **Highlighter effect implementation**:
   - Created `.highlight` utility class with angled highlighter appearance
   - 104-degree gradient for forward slash effect
   - Extended padding on all sides to ensure full text coverage
   - Applied to key concepts in boring sections
   - Replaced bold text with highlights in non-header content
   
3. **Highlighted content**:
   - Context-Compute Relationship: Key interdependence concepts
   - MCP Section: Context management and real-time retrieval capabilities
   - Agentic Coding: Parallel agent distribution and market growth
   - Current State: Productivity gains and completion time increases
   - AI Scalar: "AI doesn't replace skill; it scales it"
   - Context Section: Infinite context concept
   - Compute Section: Bottlenecks and infinite compute vision

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidlbowman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
