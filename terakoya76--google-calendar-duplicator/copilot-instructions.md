## google-calendar-duplicator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Google Apps Script (GAS) application that synchronizes calendar availability between multiple Google Calendars. Creates placeholder events (e.g., "Absence") instead of copying actual event details.

## Commands

```sh
# Setup
mise install
pnpm install

# Lint (uses Google TypeScript Style - gts)
pnpm lint

# Auto-fix lint issues
pnpm fix

# Compile TypeScript to JavaScript
pnpm compile

# Deploy to GAS (requires clasp configuration)
clasp push -P ./config/.clasp-<account>.json -A ~/.clasprc-<account>.json -I ./.claspignore
```

## Architecture

**Single-file design**: All logic in `main.ts`, compiled to `build/main.js` for GAS deployment.

**Entry points** (trigger functions for GAS time-based triggers):
- `syncEventsForToday()` - syncs current day
- `syncEventsForTommorow()` - syncs next day
- `syncEventsAfter2To14Days()` - syncs 2-14 days ahead
- `syncEventsAfter15To90Days()` - syncs 15-90 days ahead

**Core flow**:
1. `getConfig()` - reads `SYNC_CONFIG` from GAS Script Properties
2. `syncEventsForDay()` - orchestrates sync for date range
3. `deleteEventsForDay()` - removes existing placeholder events from target
4. `copyEventsForDay()` - copies events from source calendars as placeholders

**Deployment artifacts**: Only `appsscript.json` and `build/main.js` are pushed to GAS (see `.claspignore`).

## Configuration

Runtime configuration via GAS Script Properties (`SYNC_CONFIG` JSON):
- `toCalendarId`: target calendar ID
- `fromCalendarIds`: array of source calendar IDs
- `eventTitle`: placeholder event title (e.g., "Absence")

Per-account clasp configs stored in `./config/.clasp-<account>.json`.

---
> Source: [terakoya76/google-calendar-duplicator](https://github.com/terakoya76/google-calendar-duplicator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
