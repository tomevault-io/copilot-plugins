## remark-music-plugins

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Node.js project called "music-md" - a markdown renderer that converts markdown files to static HTML or PDF with music notation support. Written in plain JavaScript with TypeScript types defined using JSDoc comments.

### Core Features

- **Output formats**: Static HTML and PDF generation
- **PDF features**: Page numbering and support for manual page breaks in markdown
- **Music notation**: Two specialized markdown extensions for musical content

### Markdown Extensions

1. **LilyPond integration**:
   - Detects fenced code blocks with `lilypond` language identifier
   - Compiles LilyPond code to SVG using the LilyPond CLI
   - Embeds resulting SVG as inline images

2. **Guitar fretboard diagrams**:
   - Uses svguitar library (https://github.com/omnibrain/svguitar)
   - Fenced code blocks with `svguitar` language identifier
   - Block content contains data passed to the library's `chord` method

### Documentation

The README.md file contains comprehensive documentation explaining the markdown syntax extensions and library usage.

## Development Commands

- `npm test` - Run tests using Node.js test runner
- `npm run types` - Generate TypeScript definitions from JSDoc comments
- `npm run format` - Format code using Prettier

## Code Style

- **Language**: Plain JavaScript with JSDoc comments for TypeScript types
- **Testing**: Node.js built-in test runner
- **Formatting**: Prettier with no ending semicolons
- **Types**: TypeScript definitions generated from JSDoc comments

## Project Structure

This is a minimal Node.js repository with:

- Standard Node.js `.gitignore`
- Basic `package.json` with ISC license
- Git repository initialized

## Setup Notes

Functions should include comprehensive JSDoc comments with TypeScript type annotations. Tests should be written for the Node.js test runner. Code formatting follows Prettier configuration without semicolons.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sithmel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
