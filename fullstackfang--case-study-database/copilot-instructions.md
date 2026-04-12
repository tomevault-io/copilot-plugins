## case-study-database

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a single-page web application for managing a database of case study quotes. It's built as a standalone HTML file with inline React/Babel for simplicity - no build process required.

## Architecture

The application consists of:
- `index.html`: Self-contained React application that runs entirely in the browser
- `sample.json`: Example data structure for case study quotes
- `data/`: Directory containing JSON files with case study data

The app uses:
- React 18 (loaded via CDN)
- Babel standalone for JSX transformation in the browser
- No build tools or package manager - just open index.html in a browser

## Data Structure

Quotes have the following fields:
- `source`: The name of the case study (formerly "case" in legacy data)
- `page`: Page number (integer)
- `components`: Array of organizational components (Work, People, Structure, Culture)
- `incongruence`: Optional field for incongruence types (e.g., "Work↔Structure")
- `quote`: The actual quote text
- `notes`: Optional notes/context

## Key Features

The app supports:
- Loading data from GitHub Gist URLs
- Filtering by search term, source, components, and incongruence type
- Adding new quotes locally
- Exporting data as JSON
- Visual grouping of quotes by source with incongruence indicators

## Development

Since this is a standalone HTML file:
- No installation or build steps required
- Edit `index.html` directly to modify the application
- Test by opening index.html in any modern browser
- Data persists only in browser memory (reload loses unsaved changes)

The app fetches data from a configurable GitHub Gist URL and allows local modifications that can be exported as JSON.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FullStackFang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/FullStackFang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
