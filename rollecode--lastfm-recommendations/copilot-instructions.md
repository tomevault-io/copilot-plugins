## lastfm-recommendations

> - Never use commas or dots as thousand separators, use full numbers like 10000 and 20000

## Commits and code style

- Never use commas or dots as thousand separators, use full numbers like 10000 and 20000
- Use dots as decimal separators
- Never use ISO dates (2026-03-10) in human readable labels in UI unless programmatically required. Use Finnish format without leading zeros: d.m.yyyy (e.g. 10.3.2026)
- Always commit build and asset files
- One logical change per commit
- Keep commit messages concise (one line), use sentence case
- Update CHANGELOG.md after each change
- Use present tense in commits and CHANGELOG.md
- Use sentence case for headings (not Title Case)
- Never use bold text as headings, use proper heading levels instead
- Always add an empty line after headings
- No formatting in CHANGELOG.md except `inline code` and when absolute necessary
- Use `*` as bullets in CHANGELOG.md
- Never use Claude watermark in commits (FORBIDDEN: "Co-Authored-By")
- No emojis in commits or code

## Changelog

- Always update CHANGELOG.md, check today's date
- Never use formatting in changelog
- Use present tense in changelog
- Be concise, no repetition, one line per change
- Always use changelog formatting: ### x.x.x: yyyy-mm-dd with headings, no main "Changelog" heading, no sub headings
- Always use * as bullets in changelog
- One version per day: all changes on the same day go under one version number

## Development

- Use `npm run dev` for development server (port 3000)
- Use `npm run build` for production build
- Use `npm run start` for production server
- All Last.fm API calls go through server-side API routes in `src/app/api/`
- Environment variables: `LASTFM_API_KEY`, `LASTFM_USERNAME` in `.env` (gitignored)
- Cache directory: `cache/` at project root (gitignored, auto-created)
- Recommendation engine lives in `src/lib/lastfm.ts`
- UI components in `src/components/`, pages in `src/app/`

## Tech stack

- Next.js 15 with App Router
- TypeScript
- Tailwind CSS 4
- Last.fm Audioscrobbler API

## Claude Code workflow

- ALWAYS use Helsinki timezone (Europe/Helsinki) for all timestamps
- NEVER add Finnish language in anywhere unless the feature requires it
- Always add tasks to the Claude Code to-do list and keep it up to date
- Review your to-do list and prioritize before starting
- Prefer CSS classes over inline `style=""` attributes. Only use inline styles for truly dynamic values
- Prefer DRY code - avoid repeating logic, extract shared patterns

## Commits

- Never add Claude watermark
- Always concise, single line commits
- Always commit ALL files including build files (git add -A)
- Always run `git status` after committing to verify nothing is left uncommitted

---
> Source: [rollecode/lastfm-recommendations](https://github.com/rollecode/lastfm-recommendations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
