## dictionary-vcf

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a dictionary project for macOS voice input that generates vCard (.vcf) files to improve Japanese voice recognition. The system exploits how macOS voice input uses contacts from the Contacts app to create custom dictionaries for technical terms and programming vocabulary.

## Architecture

- **React Router v7 SPA**: Frontend application built with React Router for dictionary selection and vCard generation
- **Dictionary Files**: CSV files in `dictionaries/` containing word-pronunciation pairs
- **Build Output**: vCard format files for import into macOS Contacts

### Key Components

- `dictionaries/`: Contains categorized CSV files with technical vocabulary
- `app/`: React Router application for dictionary management
- `docs/`: Documentation about dictionary format and vCard output format
- Output generates vCard format with `FN` (full name) and `X-PHONETIC-LAST-NAME` fields

## Development Commands

```bash
# Start development server
npm run dev

# Build for production (configured for GitHub Pages deployment at /dictionary.vcf/)
npm run build

# Type checking
npm run typecheck

# Preview production build
npm run preview

# Start production server
npm start
```

## Dictionary Management

- For dictionary file guidelines, naming conventions, and content policies: `dictionaries/README.md`
- Dictionary files are located in `dictionaries/` as CSV files with format `{category}-{domain}.csv`

## Output Format

- For vCard format specification and examples: `docs/vcf-format.md`
- The application generates vCard files for import into macOS Contacts to improve voice recognition

## Deployment

Production builds are configured for GitHub Pages deployment with base path `/dictionary.vcf/` (see vite.config.ts).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pokutuna) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
