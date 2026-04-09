## swr-dotnet

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Swr.Net**

A .NET library that brings the stale-while-revalidate (SWR) caching strategy — popularized by React SWR — to the C# ecosystem. Swr.Net provides idiomatic C# APIs for Blazor, ASP.NET Core, and MAUI applications, delivering instant UI with cached data while revalidating in the background. It's a NuGet package that .NET developers can drop into any project to get smart caching with minimal ceremony.

**Core Value:** Serve stale cached data instantly while transparently revalidating in the background — the user always sees data fast, and it's always eventually fresh.

### Constraints

- **Target framework**: .NET 8+ (LTS, widest adoption across Blazor/ASP.NET Core/MAUI)
- **HTTP dependency**: v1 depends on `HttpClient` directly. Interface designed so v2 can introduce pluggable fetchers without breaking changes
- **No planning docs in git**: `.planning/` stays local-only, only code changes committed
- **Idiomatic C#**: Follow .NET conventions (async/await patterns, DI extensions, `IOptions<T>`, `ILogger<T>`, etc.) — not a direct port of JavaScript patterns
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- TypeScript - Latest - Used for Astro configuration and build-time code
- Markdown - Latest - Used for all documentation content
- JavaScript - Latest - JavaScript modules for Astro runtime and tooling
## Runtime
- Node.js 22.12.0+ (required by @astrojs/mdx)
- npm - Latest
- Lockfile: `docs/package-lock.json` (present)
## Frameworks
- Astro 6.0.1 - Static site generation and documentation hosting
- @astrojs/starlight 0.38.2 - Documentation theme built on Astro
- @astrojs/mdx 5.0.3 - MDX/Markdown support
- @astrojs/sitemap 3.7.2 - Sitemap generation
- sharp 0.34.2 - Image processing
## Key Dependencies
- @astrojs/starlight 0.38.2 - Complete documentation framework with search, navigation, and theming
- astro 6.0.1 - Core static site generator
- @astrojs/markdown-remark 7.1.0 - Markdown parsing and transformation with remark/rehype
- @astrojs/mdx 5.0.3 - MDX content support
- @pagefind/default-ui 1.3.0 - Search interface
- pagefind 1.3.0 - Site search indexing
- remark-gfm 4.0.1 - GitHub flavored markdown support
- remark-smartypants 3.0.2 - Smart typography
- shiki 4.0.0 - Syntax highlighting
- i18next 23.11.5 - Internationalization support
- astro-expressive-code 0.41.6 - Enhanced code block rendering
- rehype and rehype-* (rehype-raw, rehype-stringify, rehype-format) - HTML AST transformation
- remark and remark-* - Markdown AST tooling
- vfile 6.0.3 - Virtual file abstraction
- unified 11.0.5 - Syntax tree processing
- hast-util-* - HTML AST utilities
- mdast-util-* - Markdown AST utilities
- unist-util-* - Universal syntax tree utilities
## Configuration
- No environment variables required for documentation site
- Build-time configuration only
- `docs/astro.config.mjs` - Main Astro configuration
- `docs/tsconfig.json` - TypeScript configuration extending astro/tsconfigs/strict
- `docs/package.json` - npm scripts for dev, start, build, preview
## Platform Requirements
- Node.js 22.12.0 or higher
- npm 10.x or higher
- Static hosting (GitHub Pages)
- No runtime environment beyond static file serving
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Project Overview
## Naming Patterns
- Documentation files: lowercase with hyphens (`index.md`, `introduction.md`)
- Configuration files: camelCase or dot-notation (`astro.config.mjs`, `content.config.ts`, `tsconfig.json`)
- Asset files: lowercase with descriptive names (`houston.webp`)
- Documentation sections: lowercase with hyphens (`getting-started`, `docs`)
- Source directories: lowercase (`src`, `public`, `assets`, `content`)
- Hidden directories: dot-prefixed (`.astro`, `.vscode`, `.git`)
- Front-matter keys: kebab-case or camelCase as appropriate (e.g., `title`, `description`, `tagline`)
## Code Style
- Tool: No explicit formatter configured; follows Astro conventions
- Indentation: 2 spaces (observed in config files)
- Line endings: LF (Git configured for Unix-style)
- Location: `docs/src/content.config.ts` and `docs/astro.config.mjs`
- Format: Uses `// @ts-check` for strict type checking
- Style: Follows Astro/ESM conventions with imports and exports
- Front matter: YAML format at top of file
- Structure: Metadata first, then content
- Headings: Start with `##` (H2) after front matter
## Import Organization
## TypeScript Configuration
- Extends: `astro/tsconfigs/strict`
- Include: `.astro/types.d.ts` and all source files
- Exclude: `dist` directory
- Strict mode enabled for enhanced type safety
## Documentation Structure
- Required: `title`, `description`
- Optional: `template`, `hero`, `sidebar`, `tags`
- Meta fields snake_case with hyphens (e.g., `tagline`, `actions`, `right-arrow`)
## Section Heading
## Configuration Files
- Site: GitHub pages base URL (`https://kidchenko.github.io`)
- Base path: `/swr-dotnet`
- Integration: Starlight for documentation site
- Social links: GitHub repository
- Sidebar: Defined as static array of collections
- Uses Astro content collections API
- Loader: `docsLoader()` from Starlight
- Schema: `docsSchema()` from Starlight
- Single collection: `docs`
## VSCode Configuration
- Primary: `astro-build.astro-vscode`
- No unwanted extensions configured
- Astro debug server configured
- Program: `${workspaceFolder}/docs/node_modules/.bin/astro`
- Arguments: `dev` for development mode
## Path Aliases
- `astro/config`
- `astro:content`
- `@astrojs/starlight` and submodules
## Comments
- Front matter serves as structured metadata
- No inline code comments observed (minimal code)
- Markdown content is self-documenting
## Error Handling
## Module Design
- Single entry point: `docs/src/content.config.ts` for content routing
- Configuration centralized in root of docs directory
- Content organized by section under `src/content/docs/`
- Assets in `src/assets/`
- Public files in `public/`
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## Pattern Overview
- Content-driven architecture centered on Markdown/MDX documentation
- Framework-agnostic static site generation using Astro with Starlight integration
- Pre-built design system and navigation handling via Starlight
- Single-site deployment targeting GitHub Pages
## Layers
- Purpose: Define and organize documentation content as Markdown/MDX files
- Location: `/Users/kidchenko/kidchenko/projects/swr-dotnet/docs/src/content/docs/`
- Contains: Content collections, documentation pages, tutorials
- Depends on: Starlight schema validation via `content.config.ts`
- Used by: Astro build system to generate static HTML pages
- Purpose: Define site metadata, navigation structure, build settings, and integration configuration
- Location: `/Users/kidchenko/kidchenko/projects/swr-dotnet/docs/`
- Contains: `astro.config.mjs` (Astro configuration), `content.config.ts` (content schema), `tsconfig.json` (TypeScript settings)
- Depends on: Package dependencies defined in `package.json`
- Used by: Build system and development server
- Purpose: Store static assets including images, favicons, and other media
- Location: `/Users/kidchenko/kidchenko/projects/swr-dotnet/docs/src/assets/`, `/Users/kidchenko/kidchenko/projects/swr-dotnet/docs/public/`
- Contains: SVG favicons, image files
- Depends on: None (static files)
- Used by: Content pages for embedding images and assets
- Purpose: Transform content and assets into static HTML, CSS, and JavaScript
- Location: Node modules managed by package manager
- Contains: Astro core, Starlight theme, Sharp image processor
- Depends on: Node.js runtime, npm packages
- Used by: Development server and production build process
## Data Flow
## Key Abstractions
- Purpose: Organized grouping of documentation files with enforced schema
- Examples: `docs` collection defined in `content.config.ts` containing all markdown documentation
- Pattern: Astro Content Collections API with Starlight's pre-built schema
- Purpose: Provides pre-configured theme, navigation, and documentation components
- Examples: `@astrojs/starlight` integration in `astro.config.mjs`
- Pattern: Framework integration pattern - Starlight handles UI, layout, and user experience
- Purpose: Define site behavior, navigation structure, and metadata
- Examples: `sidebar` configuration in `astro.config.mjs` defining "Getting Started" section
- Pattern: Declarative configuration in `astro.config.mjs`
## Entry Points
- Location: `npm run dev`
- Triggers: Developer running dev command
- Responsibilities: Start local development server, watch files, enable hot reload
- Location: `npm run build`
- Triggers: CI/CD pipeline or manual build
- Responsibilities: Generate production-ready static site in `/Users/kidchenko/kidchenko/projects/swr-dotnet/docs/dist/`
- Location: `npm run preview`
- Triggers: Pre-deployment verification
- Responsibilities: Serve built static site locally for testing before deployment
- Location: `docs/src/content/docs/index.mdx` and subdirectories
- Triggers: Developer adds/edits documentation files
- Responsibilities: Provide site homepage and documentation structure
## Error Handling
- Schema validation on content files via `docsSchema()` - invalid content structure fails at build time
- TypeScript checking via `tsconfig.json` for configuration files
- Missing frontmatter or invalid YAML prevents page generation
## Cross-Cutting Concerns
- Base URL: `/swr-dotnet` (GitHub Pages subdirectory)
- Site URL: `https://kidchenko.github.io`
- Theme: Starlight default theme
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kidchenko)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kidchenko)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
