## cpabear-book

> This file describes the project structure, conventions, and workflows for AI agents working on the **CPA Bear Book** repository.

# AGENTS.md

This file describes the project structure, conventions, and workflows for AI agents working on the **CPA Bear Book** repository.

## Project Overview

CPA Bear Book is an open-source, Docusaurus-based textbook for CPA exam candidates. The site is published at [book.cpabear.com](https://book.cpabear.com) and covers three currently active exam sections:

| Section | Full Name |
|---------|-----------|
| FAR | Financial Accounting and Reporting |
| AUD | Auditing and Attestation |
| REG | Regulation |

Three additional sections (ISC, TCP, BAR) are listed on the home page but not yet implemented.

## Tech Stack

- **Framework:** [Docusaurus v3](https://docusaurus.io/) (React, TypeScript, MDX)
- **Language:** TypeScript (`~5.6`)
- **Node.js:** ≥ 20.0
- **Package manager:** npm (use `npm ci` for installs, not `yarn`)
- **Math rendering:** KaTeX via `remark-math` + `rehype-katex`
- **Diagrams:** Mermaid via `@docusaurus/theme-mermaid`
- **Icons:** `lucide-react`

## Repository Structure

```
cpabear-book/
├── content/                  # Raw source material (outlines, Q&A CSV, topic checklists)
├── docs/                     # All site documentation (MDX files)
│   ├── far/                  # Financial Accounting and Reporting pages
│   ├── aud/                  # Auditing and Attestation pages
│   └── reg/                  # Regulation pages
├── src/
│   ├── components/
│   │   ├── JournalEntry/     # Custom component to render accounting journal entries
│   │   └── SectionCard/      # Card component used on the home page
│   ├── css/
│   │   └── custom.css        # Global custom CSS
│   ├── pages/
│   │   ├── index.tsx         # Home page
│   │   └── about.tsx         # About page
│   └── theme/
│       └── CodeBlock/        # Swizzled CodeBlock to intercept `journal` language fences
├── static/                   # Static assets (images, favicon)
├── docusaurus.config.ts      # Site configuration (title, navbar, footer, plugins)
├── sidebars.ts               # Sidebar structure for FAR, AUD, and REG
├── tsconfig.json             # TypeScript configuration
└── package.json
```

## Development Commands

All commands are run from the repository root.

| Command | Purpose |
|---------|---------|
| `npm ci` | Install dependencies (use this, not `npm install`) |
| `npm run start` | Start local dev server at `http://localhost:3030` |
| `npm run build` | Production build (runs broken-link check) |
| `npm run typecheck` | Run TypeScript type checking |
| `npm run clear` | Clear Docusaurus cache |
| `npm run serve` | Serve the production build locally |

**Validate changes with:** `npm run build && npm run typecheck`

> ⚠️ `npm run build` will fail if any internal links are broken (`onBrokenLinks: "throw"`). Always verify links are valid before committing.

## Content Authoring

### Docs Location

All content lives under `docs/`. Each section has its own subdirectory mirroring the sidebar defined in `sidebars.ts`. When adding a new page:

1. Create the MDX file in the correct `docs/<section>/` subdirectory.
2. Add the doc ID (file path without extension, relative to `docs/`) to `sidebars.ts` in the appropriate category.

### Journal Entry Code Blocks

Use the custom `journal` language fence in MDX to render a formatted accounting journal entry table. The `JournalEntry` component is automatically invoked by the swizzled `CodeBlock`.

**Syntax:**

```
```journal
[Optional Date]
Dr. Account Name[accountType] amount
Cr. Account Name[accountType] amount
```​
```

**Account type codes** (optional, appended in `[brackets]`):

| Code | Type |
|------|------|
| `a` | Asset |
| `l` | Liability |
| `e` | Equity |
| `ca` | Contra-asset |
| `cl` | Contra-liability |
| `ce` | Contra-equity |

**Example:**

```
```journal
Jan 1, 2025
Dr. Cash[a] 10,000
Cr. Common Stock[e] 10,000
```​
```

### Math

Use standard LaTeX syntax. Inline: `$formula$`. Block: `$$formula$$`.

### Diagrams

Use standard Mermaid syntax in ` ```mermaid ` fences.

## Sidebar Configuration

Sidebars are defined in `sidebars.ts` and exported as `farSidebar`, `audSidebar`, and `regSidebar`. Each sidebar entry references a doc by its ID (the file path relative to `docs/`, without the `.md` extension).

## Deployment

The site deploys automatically to GitHub Pages when changes are pushed to `main` (see `.github/workflows/deploy.yml`). There is also a `test-deploy.yml` workflow for non-main branches.

## TypeScript

- The `tsconfig.json` extends `@docusaurus/tsconfig`.
- Run `npm run typecheck` to check types without building.
- All source files in `src/` should be TypeScript (`.ts` / `.tsx`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/subwaymatch)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/subwaymatch)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
