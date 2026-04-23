## huginn

> Huginn monitors Hacker News, GitHub, Reddit, and Arxiv for topics the user cares about. It collects posts, classifies them using a local AI model (Ollama), generates daily summaries, and sends alerts via Telegram. Everything runs locally, no cloud.

# Huginn

## What this project is

Huginn monitors Hacker News, GitHub, Reddit, and Arxiv for topics the user cares about. It collects posts, classifies them using a local AI model (Ollama), generates daily summaries, and sends alerts via Telegram. Everything runs locally, no cloud.

The user describes their interests in plain language in `config.json`. The system reads new content, decides what's relevant, and delivers a daily briefing plus real-time alerts.

## Tone and style rules

All text that users see (README, CLI output, help text, error messages, Telegram messages) must follow these rules:

- **Write for someone who has never heard of this tool.** No jargon from the codebase. No internal names. No assumptions about what they know.
- **Start from the pain, show the result.** Don't describe features. Describe the problem the user has and how this solves it.
- **Plain language.** "It checks if Ollama is running" not "It performs an availability probe against the Ollama inference endpoint." If a 15-year-old couldn't understand the sentence, rewrite it.
- **Short sentences.** One idea per sentence. Break long explanations into steps.
- **No marketing words.** No "powerful", "seamless", "robust", "cutting-edge", "next-generation." Just say what it does.
- **Errors must say what to do.** Not "SQLITE_NOTADB". Instead: "Database file is corrupted. Delete data/db and restart to rebuild."
- **CLI help must be complete.** Every command, what it does, one line each. A user should never need to read source code to understand a flag.

## Code organization

```
src/
  index.js              Entry point. CLI argument parsing, the collect-analyze-deliver cycle, logo.
  config.js             Loads config.json + secrets.json, applies defaults, validates keys.
  logger.js             Copies console output to data/huginn.log.
  db.js                 SQLite: table creation, migrations, all database queries.
  ollama.js             Raw HTTP client for Ollama API. No model logic.
  connectors/
    index.js            Registry: auto-discovers connectors, exports active one from config.
    qwen-3.5-9b.js      Qwen 3.5 9B connector (defaults, <think> stripping).
    gemma4-e4b.js       Gemma 4 E4B connector (defaults, thinking channel parsing).
    README.md           Architecture reference for adding new connectors.

  collector.js          Fetches Hacker News stories and comments (Algolia API).
  github-collector.js   Searches GitHub repos by topic, checks trending, monitors watched repos.
  reddit-collector.js   Reads Reddit posts via RSS feeds.
  arxiv-collector.js    Fetches academic papers from Arxiv.

  analyzer.js           Classifies stories as relevant/adjacent/irrelevant. Summarizes articles.
  github-analyzer.js    Classifies GitHub repositories the same way.
  comments.js           Analyzes comments on relevant stories for insights and engagement opportunities.
  people.js             Builds profiles of frequent commenters (SQL only, no AI).

  intelligence.js       Generates: daily briefings, weekly trends, rising alerts, opportunity alerts,
                        thread reply detection, GitHub watch alerts, competitive checks.
  delivery.js           Formats Telegram messages and writes markdown files.
```

## Configuration files

- `config.json` — all user settings (interests, sources, thresholds). Gitignored. Users copy from `config.example.json`.
- `secrets.json` — Telegram token, GitHub token. Gitignored. Users copy from `secrets.example.json`.
- `config.example.json` — template with placeholder values. Committed.
- `secrets.example.json` — template with placeholder values. Committed.

## Changelog and versioning

**This repo uses a changelog. It is mandatory.**

`CHANGELOG.md` lists every version with what changed. Every release gets an entry. No exceptions.

**"Bump" means:** update the version in `package.json` AND add a new section to `CHANGELOG.md`. Both. Always together.

Versioning: semver. Breaking changes = major. New features = minor. Fixes = patch.

## Change checklist

**After ANY code change, check these:**

1. Does `--help` text still match the actual behavior? If you added a flag, renamed something, or changed a default, update the help text in `index.js`.
2. Does `--test` still check all services the app uses? If you added a new data source, add a connectivity check.
3. Does the README still match? If you changed commands, config fields, or setup steps, update README.md.
4. **Config changes require BOTH files.** Any change to config structure MUST be reflected in BOTH `config.json` AND `config.example.json`. No exceptions. Same rule applies to `secrets.json` and `secrets.example.json`.
5. Are there hardcoded personal references? No usernames, repo names, project names, or topics in source code. Everything comes from config.
6. Does `CHANGELOG.md` have an entry for this change? If this is a release, bump `package.json` version too.

## How to add a new data source

1. Create `src/new-collector.js` with a `collect()` function
2. Store data using existing `db.upsertStories()` (for posts/articles) or add new tables in `db.js`
3. Items stored as stories are automatically picked up by the classification pipeline
4. Wire the collector into the collect phase in `index.js`
5. Add a connectivity check to `--test` in `index.js`
6. Update README, help text, and config.example.json

## How to add a new alert type

1. Add detection logic in `intelligence.js`
2. Add message formatting in `delivery.js`
3. Wire into the intelligence phase in `index.js`
4. Update README and help text

## Database and migrations

Single SQLite file at `data/db`. Schema is managed by a versioned migration system in `db.js`.

**How migrations work:** The `MIGRATIONS` array in `db.js` contains numbered migrations. On startup, the app checks which have been applied (tracked in `_migrations` table) and runs any new ones. Users never run migrations manually.

**Adding a schema change:** Add a new entry to the `MIGRATIONS` array with the next version number and the SQL to run. The baseline (version 1) creates all tables. Future migrations use `ALTER TABLE` or `CREATE TABLE` for additions.

```js
// Example: adding a column in a future version
{ version: 2, name: "add_story_language", up: "ALTER TABLE stories ADD COLUMN language TEXT" },
```

Key tables:

- `stories` — posts from all sources (HN, Reddit, Arxiv). Column `type` distinguishes them.
- `comments` — discussion comments (currently HN only).
- `story_analysis` — classification results per story (relevant/adjacent/irrelevant, summary, tags).
- `comment_analysis` — flagged comments (insight, need, opportunity).
- `github_repos` — discovered repositories.
- `github_repo_analysis` — repo classification results.
- `work_queue` — retryable work items. Failed tasks retry automatically next cycle.
- `deliveries` — generated briefings and alerts with sent/unsent tracking.
- `cursors` — progress tracking. Each collector and intelligence product has a cursor.
- `point_snapshots` — historical vote counts for detecting rising content.

## Design principles

1. **Stop and restart anytime.** All progress is in the database. Nothing in memory.
2. **If something fails, try again later.** Ollama down? Collect data, analyze when it's back. Telegram down? Save to file, send when it returns.
3. **Catch up, don't skip.** Offline for a week? Generate 7 daily briefings, one per day, not a combined blob.
4. **Missing config means skip, not crash.** No Reddit subreddits configured? Don't monitor Reddit. No Telegram token? Save to files. No GitHub token? Use lower API limits.
5. **Secrets never in committed files.** Config.json has behavior. Secrets.json has tokens. Only secrets.json is gitignored... actually both are gitignored. Example files are committed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krzysztofdudek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
