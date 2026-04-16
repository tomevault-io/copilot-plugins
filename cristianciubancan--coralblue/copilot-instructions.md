## coralblue

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands
- Run build for Windows: `.\build.cmd` or `.\build.ps1`
- Run build for Unix/Mac: `./build.sh` 
- Run with clean metadata: `./build.ps1 -cleanMetadata`
- Run with custom port: `./build.ps1 -port 8080`
- Clean dependencies only: `./build.ps1 -cleanOnly`

## Documentation Structure
- `/articles/` - Main documentation content
- `/api/` - API documentation (auto-generated)
- `toc.yml` - Table of Contents configuration file
- `index.md` - Main entry point for each section
- `docfx.json` - Configuration for the DocFX generator

## Style Guidelines
- Use Markdown for all documentation
- Filenames should be lowercase, alphanumeric with hyphens for spaces
- Headings use `#` syntax with a single space after
- Category organization: place related articles in a common directory
- Always update the appropriate `toc.yml` file when adding new content
- Keep titles concise and descriptive
- Do not include GTA:V modding content (belongs in wiki instead)
- Follow existing formatting patterns in similar documents

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CristianCiubancan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
