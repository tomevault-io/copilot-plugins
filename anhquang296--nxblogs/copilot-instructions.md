## nxblogs

> - Next.js 16 + React 19 + TypeScript + Nextra 4.6 (nextra-theme-blog)

# Tech Stack

- Next.js 16 + React 19 + TypeScript + Nextra 4.6 (nextra-theme-blog)
- Tailwind CSS 4 + shadcn/ui (new-york style) + Motion + Tabler Icons + Lucide Icons
- Yarn as package manager

# Development

- `yarn dev` — dev server with Turbopack
- `yarn build` — production build (includes Pagefind search index via postbuild)

# Project Structure

- `src/app/[lang]/` — App Router with i18n dynamic segments
- `src/components/` — React components (`ui/` for shadcn primitives, `animated-icons/` for motion icons, feature dirs like `redis-sorted-set-simulator/`)
- `src/lib/` — Utilities (`format-date.ts`, `get-posts.ts`, `get-tags.ts`)
- `src/styles/globals.css` — Theme variables and global styles
- `src/mdx-components.tsx` — Custom MDX component registry
- `content/{en,vi}/posts/` — Blog content per locale (MDX)
- `public/` — Static assets (SVGs, images, WebP)

# Content Conventions

- Blog posts live in `content/{lang}/posts/*.mdx`
- Frontmatter fields: `title`, `date` (ISO format), `tags` (string array), `description`, `enableComment` (boolean)
- Post titles must lead with the article's topic (e.g., `Sequential Read: ...`, `PostgreSQL Index: ...`), never a generic prefix like "Deep Dive:"
- Post filenames use kebab-case
- Every post must exist in both `en/` and `vi/` directories
- When creating a new blog post, write the Vietnamese version first (`content/vi/posts/`), finalize it, then create the English translation (`content/en/posts/`)
- Available MDX components: `<Posts />`, `<Tags />`, `<Avatar />`, and feature-specific components registered in `src/mdx-components.tsx`
- When mentioning a technical concept or term (e.g., data shards, parity shards, erasure coding), always briefly explain what it is so the reader can understand without prior knowledge
- For emphasis, use bold (`**text**`) only — never italic (`*text*` or `_text_`). Default to plain text; bold is not a substitute for plain prose. Bold only: (1) a list item's leading label, (2) a key term on first definition (max one per section), (3) a single pivot word or critical warning. Never bold whole phrases or sentences; aim for ≤1 bold per paragraph. To emphasize multiple items, use a list, not bold
- Avoid arrow and symbol shorthand in prose (e.g., `↔`, `->`, `→`, `<-`, `=>`, `<=>`). Spell the relationship out in words instead: write "the client sends a request to the server, and the server responds" rather than "client ↔ server"; write "a cache miss leads to a database read" rather than "cache miss -> DB read". This keeps prose readable and accessible to screen readers. Exception: arrows inside code blocks, code spans, or diagrams are fine where they carry real syntactic meaning (e.g., TypeScript `=>`, shell pipes)

# Code Style

- Prettier: no semicolons, single quotes, 120 print width, 2-space indent
- Never include comments in code — code should be self-explanatory through clear naming
- Functional components only; use `'use client'` directive for client components
- Path alias: `@/*` maps to `./src/*`
- Prefer named imports; use default imports only for components

# Naming Conventions

- Components: `PascalCase.tsx` (e.g., `AnimatedThemeSwitch.tsx`)
- Utilities: `kebab-case.ts` (e.g., `format-date.ts`)
- MDX content files: `kebab-case.mdx`
- Constants: `UPPER_SNAKE_CASE`
- Types/Interfaces: `PascalCase`

# Styling

- Tailwind CSS 4 via PostCSS plugin (no separate tailwind.config file)
- Theme variables use OKLCH color space, defined in `src/styles/globals.css`
- Dark mode via `.dark` class with CSS variable overrides
- shadcn/ui components live in `src/components/ui/`
- Custom container class: `.x\:container` (max-width 700px)

# Internationalization

- Locales: `en` (default), `vi`
- Content duplicated per locale under `content/{lang}/`
- UI text uses conditional rendering: `lang === 'vi' ? '...' : '...'`
- Locale persisted in localStorage + cookies; detection handled by `src/proxy.ts`

# Diagrams & Visualization

- Use the `excalidraw-diagram-generator` skill to create diagrams (invoke via `/excalidraw-diagram-generator`)
- Save `.excalidraw` files to the `public/` directory
- Reference diagrams in MDX using the `<ExcalidrawDiagram>` component: `<ExcalidrawDiagram src="/filename.excalidraw" alt="Description" />`
- All text content inside diagrams must be written in English
- Never generate raw SVG files or ASCII art for diagrams

## AWS Architecture Diagram Style

When creating AWS architecture diagrams, follow the official AWS reference architecture style:

- **Icon without duplicate name**: AWS icons already contain the service name visually. Do NOT add a text label that repeats the service name (e.g., don't add "CloudFront" label under the CloudFront icon). Only add a label if it describes the icon's **role or purpose** in the diagram (e.g., "CDN Cache", "Private Bucket"). Never wrap individual services in colored rectangle boxes
- **Grouping containers**: Use dashed or solid border rectangles (no fill or very light fill) to group related services (e.g., "Customer's AWS Account", "VPC", "Authentication"). Add a group label at the top-left
- **Numbered step circles**: Use small filled dark circles (#1e1e1e) with white text numbers to indicate flow order. Place them near the arrow or service they describe
- **Arrows**: Use simple solid dark arrows (#1e1e1e) to connect services. Keep arrows straight (horizontal/vertical) where possible; use angled arrows only when necessary
- **Layout**: Arrange services in a clean grid. Flow generally goes left-to-right or top-to-bottom
- **No colored backgrounds on services**: The diagram background is white; only grouping containers may have a subtle border
- **Service naming**: Use official AWS service names (e.g., "Amazon DynamoDB" not "DynamoDB", "AWS Lambda" not "Lambda") — but only in text labels that describe purpose, not as duplicate icon names
- **AWS icon library**: Icons are available at `.claude/skills/excalidraw-diagram-generator/libraries/aws-architecture-icons/`. Use the `add-icon-to-diagram.py` script. Only pass `--label` when the label describes a role/purpose (e.g., `--label "Private Bucket"`), NOT to repeat the service name

# Blog Writing

- Use the `blog-writer` skill to write blog posts (invoke via `/blog-writer`)
- Workflow: Input Gathering → Research & Outline → Vietnamese Writing → English Translation → Dual-Perspective Review
- Code examples in blog posts must always use Node.js + TypeScript

# Git Conventions

- Conventional commits: `feat:`, `fix:`, `refactor:`
- Commit messages in English

---
> Source: [anhquang296/nxblogs](https://github.com/anhquang296/nxblogs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
