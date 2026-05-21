## obsidian-limitless-google-calendar

> - Always use `npm run build-with-version` instead of `npm run build`

# Claude Code Notes for Obsidian Limitless Google Calendar Plugin

## Build Commands
- Always use `npm run build-with-version` instead of `npm run build`
- This will increment the version numbers in package.json and manifest.json automatically

## Project Structure
- Main plugin functionality is in `src/main.ts`
- Utility functions are in `src/utils/` directory
- Frontend components are in standalone TS files in the `src/` directory

## Frontmatter Structure
The plugin now supports frontmatter in generated notes with the following fields:
- `last_update_time`: ISO timestamp of when the data was fetched
- `pages_processed`: Number of API pages processed from Limitless API
- `calendar_entries_found`: Number of calendar events with transcripts
- `lifelog_entries_found`: Number of standalone lifelog entries
- `title`: Note title based on the date (format: MM-DD-YYYY Notes)

---
> Source: [techyogi/obsidian-limitless-google-calendar](https://github.com/techyogi/obsidian-limitless-google-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
