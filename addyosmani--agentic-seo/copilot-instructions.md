## agentic-seo

> ExampleDocs is a documentation site for a REST API service. The codebase contains HTML documentation pages alongside Markdown source files.

# AGENTS.md

## Project Overview

ExampleDocs is a documentation site for a REST API service. The codebase contains HTML documentation pages alongside Markdown source files.

## Project Structure

```
good-site/
  index.html              # Landing page and overview
  robots.txt              # Crawler permissions
  llms.txt                # LLM-optimized site summary
  agent-permissions.json  # Agent interaction policies
  skill.md                # Agent skill definition
  docs/
    getting-started.html  # Quickstart guide (HTML)
    getting-started.md    # Quickstart guide (Markdown)
    api/
      users.html          # Users API reference (HTML)
      users.md            # Users API reference (Markdown)
```

## Documentation Links

- API Reference: `docs/api/users.html`
- Getting Started: `docs/getting-started.html`
- Markdown sources are colocated with their HTML counterparts.

## Conventions

- All HTML pages use semantic elements (`<main>`, `<article>`, `<nav>`).
- Heading hierarchy follows H1 > H2 > H3 without skipping levels.
- Code examples use `<pre><code>` blocks with language classes.
- Every page includes `<meta name="description">` and `<link rel="canonical">`.
- Copy buttons use `class="copy-button"` and `data-copy-ai` attributes.
- Markdown versions of pages are linked via `<link rel="alternate">`.

## Development

- Static HTML; no build step required.
- Serve with any static file server for local preview.

---
> Source: [addyosmani/agentic-seo](https://github.com/addyosmani/agentic-seo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
