## iis-cms

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sanity Studio v3 for **Iloilo Integrated School Inc.** — a headless CMS managing school content (blog posts, events, FAQs, leadership profiles, alumni, admissions, site settings). Built with React 18.

## Commands

```bash
pnpm sanity dev      # Start dev server
pnpm sanity build    # Production build
pnpm sanity deploy   # Deploy to Sanity hosting
```

Package manager is **pnpm** (v9). No lint or test scripts are configured.

## Architecture

- **`sanity.config.js`** — Main config: plugins (structureTool, visionTool), custom image upload via `CompressedUploadSource`
- **`sanity.cli.js`** — CLI config pulling project ID/dataset from env vars
- **`schemas/`** — All Sanity schema definitions, registered via `schemas/index.js`
- **`components/CompressedImageInput.jsx`** — Custom image upload component that compresses images to max 1MB/1920px before upload using `browser-image-compression`

## Schema Conventions

All schemas follow these patterns:

- Use `defineType` and `defineField` from `sanity`
- Required fields use `.validation(Rule => Rule.required())`
- Image fields include `hotspot: true` and an `alt` text field in `fields`
- Rich text uses `type: 'array', of: [{type: 'block'}, {type: 'image', ...}]`
- Ordered content uses a numeric `order` field
- Slugs auto-generate from title via `source: 'title'`
- Custom `preview` selections for dashboard display

## Schema Registry

New schemas must be added to `schemas/index.js` to be registered. The index file exports an array of all schema types imported from individual schema files.

## Environment Variables

```
SANITY_STUDIO_PROJECT_ID   # Sanity project ID
SANITY_STUDIO_DATASET      # Dataset name (production)
PUBLIC_SANITY_API_VERSION   # API version date string
SANITY_AUTH_TOKEN           # For CI/CD deployment only
```

## Deployment

GitHub Actions (`.github/workflows/deploy.yml`) auto-deploys on push to `main` using `pnpm sanity deploy`. Uses Node v22.

## Git Conventions

Commit prefixes: `feat:`, `fix:`, `ci:`, `wip:` — with optional scope like `fix(ci):`.

---
> Source: [shisole/iis-cms](https://github.com/shisole/iis-cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
