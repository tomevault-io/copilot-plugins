## tuda-hivemind-quartz-test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Quartz v4 static site generator deployment for a collaborative TU Darmstadt computer science knowledge base. The framework is TypeScript/Preact-based and generates a static site from Obsidian-compatible markdown notes.

**Content is managed as a Git submodule** (`content/` → https://github.com/Juliper/TuDa_HiveMind.git)

## Common Commands

```bash
# Development server with hot reload
npx quartz build --serve

# Build for production
npx quartz build

# Type check and format validation
npm run check

# Auto-format with Prettier
npm run format

# Run tests
npm run test

# Profile build performance
npm run profile
```

## Build Pipeline

The build process (orchestrated by `quartz/build.ts`) follows this flow:

1. **Clean** - Clear `public/` output directory
2. **Glob** - Scan for markdown files in `content/`
3. **Parse** - Convert markdown to AST using remark
4. **Filter** - Apply filter plugins (e.g., `RemoveDrafts()` hides `draft: true` pages)
5. **Transform** - Apply transformers (syntax highlighting, LaTeX/KaTeX, wikilinks, etc.)
6. **Emit** - Generate HTML, assets, RSS, sitemap to `public/`

## Architecture

```
quartz/                    # Core framework (TypeScript)
├── components/            # Preact UI components (23 components)
│   └── scripts/           # Client-side interactivity (search, graph, SPA routing)
├── plugins/               # Plugin system
│   ├── transformers/      # Modify content (frontmatter, syntax, latex, links, moduleInfo)
│   ├── filters/           # Filter content (RemoveDrafts)
│   └── emitters/          # Generate output (HTML pages, RSS, sitemap, OG images)
├── processors/            # Content pipeline orchestration
├── styles/                # SCSS (custom.scss for customizations)
└── util/                  # Shared utilities

content/                   # Markdown notes (Git submodule)
public/                    # Generated static site output
```

## Key Configuration Files

- **`quartz.config.ts`** - Site title, base URL, theme colors, fonts, plugin configuration
- **`quartz.layout.ts`** - Component layout for pages (sidebars, header, footer)

## Content Format

Markdown files support:
- **Frontmatter**: `title`, `description`, `aliases`, `tags`, `draft: true`, `ects`, `difficulty_ratings`
- **Wikilinks**: `[[note]]`, `[[note|display text]]`, `[[note#heading]]`
- **Embeds**: `![[other-note]]`, `![[image.png]]`
- **Callouts**: Obsidian-style `> [!info]`, `> [!warning]`, etc.
- **LaTeX**: Inline `$...$` and block `$$...$$` (rendered with KaTeX)
- **GFM**: Tables, task lists, code blocks with syntax highlighting

## Deployment

GitHub Actions workflow (`.github/workflows/deploy.yml`) automatically:
1. Triggers on push to `v4` branch or via `repository_dispatch` from the content submodule
2. Updates content submodule to latest
3. Builds with `npx quartz build`
4. Deploys to GitHub Pages

When content is pushed to the submodule repo, it sends a `repository_dispatch` event to trigger rebuilds.

## Submodule Management

The `content/` folder is a Git submodule. To commit and push content changes:

```bash
cd content
git checkout main              # Ensure on main branch (not detached HEAD)
git add . && git commit -m "message"
git push origin main           # This triggers the deploy workflow
cd ..
git add content                # Update submodule reference in main repo
git commit -m "update content submodule"
git push origin v4
```

## Customization

- **Styles**: Add custom CSS to `quartz/styles/custom.scss`
- **Components**: Modify or add Preact components in `quartz/components/`
- **Plugins**: Extend functionality via `quartz/plugins/`
- **Layout**: Adjust component placement in `quartz.layout.ts`

## Requirements

- Node.js >= 22
- npm >= 10.9.2

---
> Source: [Juliper/TuDa_HiveMind_Quartz_Test](https://github.com/Juliper/TuDa_HiveMind_Quartz_Test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
