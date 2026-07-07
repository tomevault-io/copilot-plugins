## website

> SPDX-FileCopyrightText: 2026 James Harton

<!--
SPDX-FileCopyrightText: 2026 James Harton

SPDX-License-Identifier: Apache-2.0
-->

# AGENTS.md

This file provides guidance to coding assistants while working with this project.

## Project Overview

This is the Beam Bots documentation website, built with [Zola](https://www.getzola.org/).

## Common Commands

```bash
zola serve          # Local development server
zola build          # Build static site
pipx run reuse lint # Check REUSE license compliance
```

## Structure

- `content/` - Markdown content (blog posts, pages)
- `templates/` - Zola templates (Tera)
- `sass/` - Stylesheets
- `static/` - Static assets (images, CSS)

## Content Guidelines

- Keep hexdocs as the source of truth for API documentation; the nav "Docs" link points straight at it rather than mirroring guides on the site.
- The ecosystem is surfaced by linking to bb's [dependents on Hex](https://hex.pm/packages/bb/dependents), not a hand-maintained package list.
- The website should link out rather than duplicate content that lives on hexdocs or hex.pm.
- Blog posts for announcements and tutorials that benefit from richer formatting.

---
> Source: [beam-bots/website](https://github.com/beam-bots/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
