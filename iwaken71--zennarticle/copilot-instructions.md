## zennarticle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Zenn CLI project - a Japanese technical writing platform for developers. Articles are written in Markdown with YAML frontmatter and managed through Git.

## Common Commands

### Development
- `npm run preview` - Start local preview server at http://localhost:8000
- `npm run create` - Create a new article with boilerplate

### Article Management
Articles are stored in `/articles/` directory. Each article uses this format:
```yaml
---
title: "Article Title"
emoji: "📝"
type: "tech" # or "idea"
topics: ["topic1", "topic2"]
published: true # or false
---
```

## Project Structure

- `/articles/` - Markdown articles with frontmatter
- `/images/` - Image assets organized by topic (avp/, babylon/, gif/, hololens/, hololens-2022-2/)
- `/books/` - Book content directory (currently unused)

## Key Conventions

1. **Article naming**: Use descriptive kebab-case filenames (e.g., `babylonjs-webxr-handtracking.md`)
2. **Image references**: Store images in appropriate subdirectories under `/images/`
3. **Topics**: Articles focus on AR/VR technologies (Babylon.js, HoloLens, Unity, WebXR, etc.)
4. **Language**: Articles are written in Japanese

## Dependencies

- `zenn-cli`: ^0.1.153 - Core CLI tool for Zenn platform

## Important Notes

- No test framework is configured
- Articles can be previewed locally before publishing
- The repository is connected to GitHub for version control
- Published status is controlled via the `published` field in frontmatter

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iwaken71) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
