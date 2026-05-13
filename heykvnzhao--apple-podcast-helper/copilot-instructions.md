## apple-podcast-helper

> The core entry point is `extract-transcripts.js`, which drives transcript discovery, metadata lookup, and Markdown generation. Generated files land in the git-ignored `transcripts/` directory; clean it between runs if you want a fresh export. Keep prompt templates and LLM workflows in `prompts/` (currently `podcasts-summarizer.md`). Dependency state lives in `package.json` and `pnpm-lock.yaml`; avoid editing `node_modules/` directly. Treat the Apple Podcasts cache under `~/Library/Group Containers/243LU875E5.groups.com.apple.podcasts/` as read-only input.

# Repository Guidelines

## Project Structure & Module Organization
The core entry point is `extract-transcripts.js`, which drives transcript discovery, metadata lookup, and Markdown generation. Generated files land in the git-ignored `transcripts/` directory; clean it between runs if you want a fresh export. Keep prompt templates and LLM workflows in `prompts/` (currently `podcasts-summarizer.md`). Dependency state lives in `package.json` and `pnpm-lock.yaml`; avoid editing `node_modules/` directly. Treat the Apple Podcasts cache under `~/Library/Group Containers/243LU875E5.groups.com.apple.podcasts/` as read-only input.

## Build, Test, and Development Commands
- `pnpm install` (or `npm install`) — install `xml2js` and sync the lockfile.
- `node extract-transcripts.js --no-timestamps` — batch-export every cached transcript without timestamp markers.
- `node extract-transcripts.js path/to/file.ttml transcripts/sample.md` — run a targeted conversion while iterating on parsing changes.
- `NODE_ENV=development node extract-transcripts.js` — enable verbose logging blocks guarded by `process.env.NODE_ENV` if you add them.

## Coding Style & Naming Conventions
Match the existing Node 18 + CommonJS style: use tabs for indentation, `const` by default, and `camelCase` for variables and functions. CLI flags stay in `--kebab-case`; exported filenames should follow the existing `show-date-title.md` slug pattern (`slugify` + `truncateSlug`). Prefer small pure helpers and keep side effects concentrated around filesystem and child-process calls. Document any new environment variables in `README.md`.

## Testing Guidelines
There is no automated test suite yet; validate changes by running the batch command against a real Podcasts cache and spot-checking Markdown output. When adjusting metadata queries, verify both happy-path exports and the fallback naming branch by temporarily renaming the SQLite file. Capture sample outputs in `transcripts/` and delete them before committing. If you add structured tests later, colocate them under `__tests__/` with `.spec.js` suffixes and wire them into a `pnpm test` script.

## Commit & Pull Request Guidelines
Follow the conventional commits pattern already in history (`feat:`, `docs:`, etc.) and keep the scope concise. Each PR should describe the behavioural change, note how you validated it, and link to any related issue. Include before/after snippets or command output when modifying export formatting. Ensure generated transcripts stay untracked and that lockfiles reflect dependency edits before requesting review.

---
> Source: [heykvnzhao/apple-podcast-helper](https://github.com/heykvnzhao/apple-podcast-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
