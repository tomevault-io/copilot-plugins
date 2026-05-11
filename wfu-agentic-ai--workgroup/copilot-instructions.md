## workgroup

> > Project instructions

# WFU Agentic AI Workgroup

> Project instructions

## Workgroup overview

This is a faculty/staff workgroup at Wake Forest University focused on agentic AI tools in the CLI. It is **not** a credit-bearing course — there are no grades, no formal assessment, and membership is rolling (participants join and leave at any time).

- **Audience:** WFU faculty and staff with little or no CLI experience
- **Format:** Weekly in-person meetings + async web content
- **Facilitator:** Will Fleeson (Department of Psychology)
- **Deliverable:** A Quarto website deployed to GitHub Pages

## Learning goals

The workgroup moves participants from zero CLI knowledge toward independent use of agentic AI tools:

1. Navigate the file system and run commands in a terminal
2. Use Git and GitHub for version control and collaboration
3. Configure shell environments and dotfiles
4. Interact with Claude Code for agentic coding workflows
5. Access and configure APIs for AI services
6. Set up and use MCP servers for tool integration
7. Author reproducible documents with Markdown and Quarto

## Content guidelines

When writing tutorials, session notes, or resource pages:

- **Assume zero CLI knowledge.** Define terms on first use. Explain what commands do before showing them.
- **Progressive difficulty.** Early content covers basic navigation and file operations. Later content introduces Git, Claude Code, APIs, and MCPs.
- **Worked examples over abstractions.** Show a concrete task, walk through the steps, then generalize. Avoid leading with theory.
- **Copy-paste friendly.** Code blocks should be runnable as-is. Include expected output where helpful.
- **Platform awareness.** Most participants use macOS, but some use Windows. Tutorials should default to macOS examples and note Windows differences (e.g., PowerShell vs bash, path separators, installer commands) in callouts or inline when behavior diverges.
- **No jargon without context.** If a term is unavoidable, define it inline or link to the glossary.

## Technology stack

- **Shell:** bash (default macOS shell via Terminal.app or iTerm2)
- **Version control:** Git + GitHub
- **AI tools:** Claude Code (CLI), Anthropic API, MCP servers
- **Authoring:** Markdown, Quarto (`.qmd` files)
- **Deployment:** GitHub Actions (`.github/workflows/deploy.yml` renders via Nix + deploys to GitHub Pages on push to `main`)
- **Environment:** Nix flake for reproducible development setup

## Site structure

```
_quarto.yml            — Site configuration (navbar, theme, extensions)
_variables.yaml        — Reusable template variables (facilitator name, etc.)
glossary.yml           — Term definitions for the {{< glossary >}} shortcode
index.qmd              — Landing page (welcome, meeting info, getting started)
about.qmd              — Purpose, learning goals, format, facilitator
assets/                — Custom SCSS theme overrides
  theme-light.scss
  theme-dark.scss
_extensions/           — Quarto extensions (glossary, fontawesome)
sessions/              — Session notes, one file per meeting (date-based)
  index.qmd            — Session log, reverse chronological
  _metadata.yml        — Shared front matter defaults for sessions
tutorials/             — Self-contained topic tutorials
  index.qmd            — Tutorial catalog with difficulty indicators
  _metadata.yml        — Shared front matter defaults for tutorials
  images/              — Tutorial images (logos, screenshots)
  cli-fundamentals.qmd
  working-with-the-filesystem.qmd
  shell-configuration.qmd
  git-fundamentals.qmd
  github-collaboration.qmd
  claude-code-intro.qmd
  apis-and-mcps.qmd
resources/             — Reference material
  index.qmd            — Resource hub
  _metadata.yml        — Shared front matter defaults for resources
  cheatsheets.qmd
  tools.qmd
  troubleshooting.qmd
  glossary.qmd
specs/                 — Internal planning and progress docs (not rendered)
```

### Adding a new session

Create `sessions/YYYY-MM-DD.qmd` with front matter:

```
---
title: "Session title"
date: "YYYY-MM-DD"
date-modified: today        # Quarto auto-fills the current date on render
description: "Brief summary of what was covered."
categories: [Topic1, Topic2] # Used for listing filters on sessions/index.qmd
---
```

The listing on `sessions/index.qmd` picks up new session files automatically.

### Adding a new tutorial

Create `tutorials/topic-name.qmd` with front matter that includes `categories`, one of `"Getting started"`, `"Version control"`, or `"AI tools"`), and then subcategories as needed (e.g., `"GitHub"`, `"Claude Code"`, `"APIs"`, and so on). Also include `description` and `date` for the listing page.

```
---
title: "Tutorial title"
description: "Brief description of the tutorial."
date: "YYYY-MM-DD"
date-modified: today          # Quarto auto-fills the current date on render
categories: ["Getting started", "CLI"]  # Primary: "Getting started", "Version control", or "AI tools"; then subcategories
order: 1                      # Controls sort order within the tutorial listing
draft: true                   # Set to false when ready to publish
image: images/example.png     # Thumbnail for the listing card (optional)
---
```

The listing on `tutorials/index.qmd` picks up new tutorial files automatically.

## Quarto conventions

- **Theme:** simplex with custom light/dark SCSS overrides in `assets/` (`theme-light.scss`, `theme-dark.scss`)
- **Code blocks:** Use `code-copy: true` for copyable code blocks
- **External links:** Open in new window (`link-external-newwindow: true`)
- **Front matter:** Every `.qmd` file needs `title` at minimum; tutorials should also have `description` and `date`
- **Callouts:** Use Quarto callouts (`.callout-note`, `.callout-tip`, `.callout-warning`) for asides and important information
- **Images:** Store in an `images/` subdirectory relative to the page that uses them

## Glossary

Term definitions live in `glossary.yml` at the project root. The `maehr/glossary` Quarto extension renders inline popups via the shortcode:

```
{{< glossary CLI >}}
```

To display custom link text (e.g., plurals or alternate phrasing), use:

```
{{< glossary CLI display="command-line interfaces" >}}
```

To add a new term, append an entry to `glossary.yml`:

```
"Term Name": |
  Definition text. Keep it concise — one to three sentences aimed at beginners.
```

Terms are organized by tutorial section with comment headers (e.g., `# Filesystem concepts`).

## Template variables

Reusable values are defined in `_variables.yaml` and referenced with the `var` shortcode:

```
{{< var group.facilitator >}}
```

Current variables include `group.facilitator`, `group.facilitator_email`, and `group.facilitator_dept`.

## Coding standards

Follow the conventions in the global `~/.claude/CLAUDE.md`:

- R: tidyverse, native pipe (`|>`), 2-space indent
- Python: data science stack, ruff, 4-space indent
- Bash: POSIX-compliant, 2-space indent
- Line limit: 80 characters (except Markdown)
- Naming: snake_case (functions/variables)
- Lists: Use `-` not `*` or `+`
- Conventional commits: `type(scope): message`

## Specs directory

The `specs/` directory contains planning, progress, and implementation documents for the workgroup. These files are excluded from the rendered site (via `.gitignore` and not being listed in `_quarto.yml`). They are internal project management documents.

---
> Source: [wfu-agentic-ai/workgroup](https://github.com/wfu-agentic-ai/workgroup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
