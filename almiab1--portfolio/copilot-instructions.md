## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Development

```bash
pnpm run dev       # Start dev server at http://localhost:4321
pnpm run build     # Build to ./dist
pnpm run preview   # Preview production build
pnpm run lint      # Lint .astro,.js,.jsx,.ts,.tsx files
pnpm run format    # Format with Prettier
```

### Requirements

- **Node.js**: v22 (see `.nvmrc`)
- **Package Manager**: pnpm (NOT npm)
- **Installation**: `nvm use && pnpm install`

## Architecture Overview

### Stack

- **Framework**: Astro 5 (islands architecture)
- **UI Library**: React 19 (for interactive components)
- **Styling**: Tailwind CSS v4 + shadcn/ui utilities
- **Content**: MDX with Astro Content Collections
- **i18n**: Custom implementation (es/en)
- **SEO**: `@astrojs/sitemap` + JSON-LD schemas in `src/lib/schema.ts`
- **Deployment**: Vercel

### Key Directories

```
src/
├── components/
│   ├── blocks/      # Page sections (Hero, ProjectGrid, ContactCTA)
│   ├── core/        # Core components (Header)
│   └── ui/          # shadcn UI components
├── content/
│   └── projects/    # MDX projects organized by language (es/, en/)
├── i18n/
│   ├── ui.ts        # Translation dictionary (ALL UI text)
│   ├── utils.ts     # Astro i18n utilities
│   └── translations.ts  # React i18n helpers
├── layouts/
│   └── BaseLayout.astro  # Main layout with SEO/meta tags
├── lib/
│   ├── schema.ts    # JSON-LD structured data
│   └── i18n-content.ts  # Content collection i18n helpers
└── pages/
    ├── index.astro  # Spanish home (/)
    ├── es/          # Explicit Spanish pages
    └── en/          # English pages (/en/*)
```

### Path Aliases

Configured in `tsconfig.json`:

- `@/*` → `src/*`
- `@components/*` → `src/components/*`
- `@layouts/*` → `src/layouts/*`
- `@lib/*` → `src/lib/*`

## Internationalization (i18n) System

### Critical Rules

1. **NEVER hardcode text** in Spanish or English in components
2. **ALWAYS use translations** from `src/i18n/ui.ts`
3. **Content MUST include** `lang` and `translationKey` in frontmatter
4. **Consult `/docs/i18n/`** for detailed i18n documentation

### Configuration

```javascript
// astro.config.mjs
i18n: {
  locales: ['es', 'en'],
  defaultLocale: 'es',
  routing: {
    prefixDefaultLocale: false,     // No /es/ prefix in URLs
    redirectToDefaultLocale: false,
  },
  fallback: { en: 'es' }
}
```

### URL Structure

- Spanish (default): `/`, `/work`, `/work/[slug]`
- English: `/en`, `/en/work`, `/en/work/[slug]`

### Translating Components

**Astro Components:**

```astro
---
import { useTranslations } from '@/i18n/utils';

const lang = Astro.currentLocale || 'es';
const t = useTranslations(lang);
---

<h1>{t('home.title')}</h1>
```

**React Components:**

```tsx
import { t, type Language } from '@/i18n/translations';

function Component({ currentLocale = 'es' }: Props) {
  const translate = t(currentLocale as Language);
  return <h1>{translate('nav.home')}</h1>;
}
```

### Adding New Translations

1. Add to `src/i18n/ui.ts` for both `es` and `en`
2. Use in components via `t('key')` or `translate('key')`

### MDX Content (Projects)

Projects live in `src/content/projects/es/` and `src/content/projects/en/`.

**Spanish project** (`src/content/projects/es/mi-proyecto.mdx`):

```yaml
---
title: 'Título del Proyecto'
summary: 'Resumen breve'
date: '2024-01-01'
lang: 'es' # REQUIRED
translationKey: 'mi-proyecto' # REQUIRED for linking translations
type: 'web'
status: 'completed'
tags: ['AI', 'Web']
tech: ['Python', 'React']
role: 'Lead Engineer'
links:
  demo: 'https://demo.com'
  repo: 'https://github.com/user/repo'
cover: { src: '/og/proyecto.png', alt: 'Project view' }
---
```

**English translation** (`src/content/projects/en/my-project.mdx`):

```yaml
---
title: 'Project Title'
summary: 'Brief summary'
date: '2024-01-01'
lang: 'en' # REQUIRED
translationKey: 'mi-proyecto' # SAME KEY as Spanish version
# ... rest of fields
---
```

### Getting Content by Language

```typescript
import { getProjectsByLang } from '@/lib/i18n-content';

// In Spanish pages
const projects = await getProjectsByLang('es');

// In English pages
const projects = await getProjectsByLang('en');
```

## Coding Conventions

### Language Rules

- **Code**: English (variables, functions, comments)
- **Content**: Spanish (projects, UI text via i18n)
- **Documentation**: Spanish (docs/, README)
- **Git Commits**: Spanish preferred

### TypeScript

- Strict mode enabled
- **NO `any`** — use `unknown` or specific types
- Interfaces for objects, types for unions/aliases
- Explicit prop types required

### Naming

- Components: `PascalCase.astro` or `PascalCase.tsx`
- Utilities: `kebab-case.ts`
- Variables/functions: `camelCase`
- Constants: `UPPER_CASE`
- MDX content: `kebab-case.mdx`

### Tailwind CSS v4

- **DO NOT use `@apply`** (v4 style)
- Order classes: layout → spacing → colors → effects
- Use CSS variables for theming
- Responsive-first approach

### React Components

- Functional components with TypeScript
- Use `"use client"` directive for interactivity
- Define Props interface
- Prefer composition over inheritance

### Imports

- Use path aliases (e.g., `@/components/ui/button`)
- Order: external → internal → local
- Group by type

## Linting and Formatting

### ESLint Configuration

- Flat config format (`eslint.config.js`)
- **MUST ignore**: `dist/**`, `.astro/**`, `node_modules/**`
- React rules disabled for `*.astro` files:
  - `react/no-unknown-property: off`
  - `react/jsx-key: off`
  - `react-hooks/rules-of-hooks: off`

### Prettier Configuration

- **Tailwind v4 requires**: `tailwindStylesheet: "./src/styles/global.css"`
- **Plugin order matters**: `prettier-plugin-tailwindcss` must be last
- Plugins: `prettier-plugin-astro`, `prettier-plugin-tailwindcss`

### Before Committing

```bash
pnpm run lint      # Check for linting errors
pnpm run format    # Format code
pnpm run build     # Verify build succeeds
```

## Git Workflow

### Branch Structure

- `new-portfolio` — Active development branch (PRs target this)
- `main` — Legacy branch
- Feature branches: Created from and merged into `new-portfolio`

### Git Worktrees

- Worktrees stored in `.trees/` directory
- Clean up with `git worktree remove <path>` before deleting branch

### Creating PRs

- **Target branch**: `new-portfolio` (NOT `main`)
- Create from feature branch to `new-portfolio`

## Deployment

### Vercel Configuration

- **Framework**: Astro
- **Build Command**: `pnpm run build`
- **Output Directory**: `dist`
- **Install Command**: `pnpm install`
- **Node.js Version**: 22

### Environment URLs

| Branch      | Environment | URL                         |
| ----------- | ----------- | --------------------------- |
| `main`      | Production  | `https://alejandromira.com` |
| `develop`   | Preview     | Auto-generated by Vercel    |
| `feature/*` | Preview     | Auto-generated by Vercel    |

### DNS (OVH)

- A Record: `@` → `76.76.21.21` (Vercel)
- CNAME: `www` → `cname.vercel-dns.com`

## Documentation

The `/docs` directory contains comprehensive documentation optimized for LLM agents:

```
docs/
├── README.md                    # Main index
├── architecture/                # System architecture
│   ├── project-structure.md
│   ├── tech-stack.md
│   └── build-system.md
├── i18n/                        # i18n system (CRITICAL)
│   ├── i18n-overview.md
│   ├── i18n-content-guide.md
│   └── i18n-api-reference.md
├── features/                    # Feature docs
│   ├── components.md
│   ├── content-collections.md
│   └── seo.md
└── guides/                      # How-to guides
    ├── getting-started.md
    ├── development-workflow.md
    ├── deployment.md
    └── project-schema-guide.md
```

**ALWAYS consult** `/docs/i18n/` before modifying i18n functionality.

## SEO and Structured Data

### JSON-LD Schemas

Located in `src/lib/schema.ts`:

- `ldPerson()` — Person schema
- `ldWebsite()` — Website schema
- `ldCreativeWork(props)` — Creative work
- `ldBlogPosting(props)` — Blog post
- `ldEvent(props)` — Event schema

### Usage

```astro
---
import { ldPerson, ldWebsite } from '@/lib/schema';
---

<script type="application/ld+json" set:html={JSON.stringify(ldPerson())} />
<script type="application/ld+json" set:html={JSON.stringify(ldWebsite())} />
```

## Project Configuration Files

- `astro.config.mjs` — Astro configuration with i18n settings
- `tsconfig.json` — TypeScript with path aliases
- `eslint.config.js` — ESLint flat config
- `.prettierrc` — Prettier with Tailwind v4 support
- `vercel.json` — Vercel deployment config
- `.nvmrc` — Node.js version (v22)

## Common Pitfalls

### i18n

- ❌ Hardcoding text: `<button>Ver proyectos</button>`
- ✅ Using translations: `<button>{translate('work.viewAll')}</button>`
- ❌ Forgetting `lang` field in MDX frontmatter
- ✅ Including `lang: 'es'` and `translationKey: 'unique-key'`
- ❌ Different `translationKey` values for same project in different languages
- ✅ Same `translationKey` in both Spanish and English versions

### Tailwind

- ❌ Using `@apply` in CSS files (not supported in v4)
- ✅ Using inline utility classes

### Git

- ❌ Creating PRs to `main` branch
- ✅ Creating PRs to `new-portfolio` branch

### Package Manager

- ❌ Using `npm install` or `npm run`
- ✅ Using `pnpm install` or `pnpm run`

## Sub-Agent Workflow

## Rules

- After a plan mode phase you should create a `.claude/sessions/context_session_{feature_name}.md` with the definition of the plan
- Before you do any work, MUST view files in `.claude/sessions/context_session_{feature_name}.md` file and `.claude/doc/{feature_name}/*` files to get the full context (feature_name being the id of the session we are operate, if file doesnt exist, then create one)
- `.claude/sessions/context_session_{feature_name}.md` should contain most of context of what we did, overall plan, and sub agents will continusly add context to the file
- After you finish the work, MUST update the `.claude/sessions/context_session_{feature_name}.md` file to make sure others can get full context of what you did
- After you finish the each phase, MUST update the `.claude/sessions/context_session_{feature_name}.md` file to make sure others can get full context of what you did

## Sub-Agent Workflow

This project uses specialized sub-agents for different concerns. Always consult the appropriate agent:

- **shadcn-ui-architect**: UI building & component architecture
- **qa-criteria-validator**: Final UI/UX validation and feedback
- **ui-ux-analyzer**: UI review, improvements & tweaking
- **frontend-developer**: Client-side business logic
- **frontend-test-engineer**: Frontend test case definitions
- **typescript-test-explorer**: Test case design
- **hexagonal-backend-architect**: NextJS API & backend architecture
- **backend-test-architect**: Backend test definitions

Sub agents will do research about the implementation and report feedback, but you will do the actual implementation;
When passing task to sub agent, make sure you pass the context file, e.g. `.claude/sessions/context_session_{feature_name}.md`.
After each sub agent finish the work, make sure you read the related documentation they created to get full context of the plan before you start executing

## Code Writing Standards

- **Simplicity First**: Prefer simple, clean, maintainable solutions over clever ones
- **ABOUTME Comments**: All files must start with 2-line comment with "ABOUTME: " prefix
- **Minimal Changes**: Make smallest reasonable changes to achieve desired outcome
- **Style Matching**: Match existing code style/formatting within each file
- **Preserve Comments**: Never remove comments unless provably false
- **No Temporal Naming**: Avoid 'new', 'improved', 'enhanced', 'recently' in names/comments
- **Evergreen Documentation**: Comments describe code as it is, not its history

## Version Control

- Non-trivial edits must be tracked in git
- Create WIP branches for new work
- Commit frequently throughout development
- Never throw away implementations without explicit permission

## Testing Requirements

**NO EXCEPTIONS POLICY**: All projects MUST have:

- Unit tests

The only way to skip tests: Alex EXPLICITLY states "I AUTHORIZE YOU TO SKIP WRITING TESTS THIS TIME."

- Tests must comprehensively cover all functionality
- Test output must be pristine to pass
- Never ignore system/test output - logs contain critical information

## Architecture Compliance

When writing backend code:

1. **Keep Domain Pure**: Zero framework dependencies in `src/domain/`
2. **Define Ports First**: Interfaces in `src/application/ports/` before implementations
3. **Thin Controllers**: API routes delegate immediately to use cases
4. **Dependency Injection**: All dependencies injected via constructor
5. **Repository Pattern**: Data access only through repository interfaces

When writing frontend code:

1. **Container Pattern**: Separate business logic from presentation
2. **Custom Hooks**: Business logic in hooks (e.g., `useConversation`)
3. **Feature Organization**: Group by feature in `app/features/`
4. **Component Purity**: Components receive props, hooks manage state

## Code Writing

- YOU MUST ALWAYS address me as "Alex" in all communications.
- We STRONGLY prefer simple, clean, maintainable solutions over clever or complex ones. Readability and maintainability are PRIMARY CONCERNS, even at the cost of conciseness or performance.
- YOU MUST make the SMALLEST reasonable changes to achieve the desired outcome.
- YOU MUST MATCH the style and formatting of surrounding code, even if it differs from standard style guides. Consistency within a file trumps external standards.
- YOU MUST NEVER make code changes unrelated to your current task. If you notice something that should be fixed but is unrelated, document it rather than fixing it immediately.
- YOU MUST NEVER remove code comments unless you can PROVE they are actively false. Comments are important documentation and must be preserved.
- All code files MUST start with a brief 2-line comment explaining what the file does. Each line MUST start with "ABOUTME: " to make them easily greppable.
- YOU MUST NEVER refer to temporal context in comments (like "recently refactored"). Comments should be evergreen and describe the code as it is.
- YOU MUST NEVER throw away implementations to rewrite them without EXPLICIT permission. If you're considering this, YOU MUST STOP and ask first.
- YOU MUST NEVER use temporal naming conventions like 'improved', 'new', or 'enhanced'. All naming should be evergreen.
- YOU MUST NOT change whitespace unrelated to code you're modifying.

## Version Control

- For non-trivial edits, all changes MUST be tracked in git.
- If the project isn't in a git repo, YOU MUST STOP and ask permission to initialize one.
- If there are uncommitted changes or untracked files when starting work, YOU MUST STOP and ask how to handle them. Suggest committing existing work first.
- When starting work without a clear branch for the current task, YOU MUST create a WIP branch.
- YOU MUST commit frequently throughout the development process.

## Getting Help

- Always ask for clarification rather than making assumptions
- Stop and ask for help when stuck, especially when human input would be valuable
- If considering an exception to any rule, stop and get explicit permission from Alex first

## Testing

- Tests MUST comprehensively cover ALL implemented functionality.
- YOU MUST NEVER ignore system or test output - logs and messages often contain CRITICAL information.
- Test output MUST BE PRISTINE TO PASS.
- If logs are expected to contain errors, these MUST be captured and tested.
- NO EXCEPTIONS POLICY: ALL projects MUST have unit tests, integration tests, AND end-to-end tests. The only way to skip any test type is if Alex EXPLICITLY states: "I AUTHORIZE YOU TO SKIP WRITING TESTS THIS TIME."

## Compliance Check

Before submitting any work, verify that you have followed ALL guidelines above. If you find yourself considering an exception to ANY rule, YOU MUST STOP and get explicit permission from Alex first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/almiab1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/almiab1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
