## trash-cleaner

> CLI tool that deletes, archives, or marks-as-read trash/spam emails from Gmail and Outlook based on configurable keyword rules.

# Trash Cleaner — Development Conventions

## Overview
CLI tool that deletes, archives, or marks-as-read trash/spam emails from Gmail and Outlook based on configurable keyword rules.

## Environment
- **Node.js >= 18.0.0** required
- TypeScript source compiled via `tsx` (no separate build step)

## Build & Test

```bash
npm install
npm test                              # runs: c8 mocha --recursive
npm run lint                          # runs: eslint .
npx mocha test/trash-cleaner.spec.js  # single test file
npx mocha --grep "finds spam"         # single test by name
npm install -g .                      # install globally for manual testing
```

After completing a task, run `npm install -g .` so the latest changes are available globally for manual testing.

## Architecture

This is a Node.js CLI tool that deletes, archives, or marks-as-read trash/spam emails from Gmail and Outlook based on configurable keyword rules.

**Flow:** `bin/trash-cleaner` → `index.js` → `Cli` → `EmailClient` + `TrashCleaner`

The codebase uses a **factory pattern** throughout. `TrashCleanerFactory`, `GmailClientFactory`, and `OutlookClientFactory` each expose a `getInstance()` method that handles async setup (reading config, OAuth) before returning the usable instance.

**Key abstractions:**
- `EmailClient` (base) → `GmailClient` / `OutlookClient` — fetch unread emails + delete/archive/mark-as-read
- `ConfigStore` (base) → `FileSystemConfigStore` — read/write JSON config files from `config/`
- `ProgressReporter` (base) → `ConsoleProgressReporter` — event-based progress reporting with `ora` spinner
- `TrashRule` (base) → `KeywordTrashRule` — regex-based matching against email fields and labels

**Trash matching:** Keywords from `config/keywords.json` become `KeywordTrashRule` objects. Each rule has a regex `value`, target `fields` (from/subject/snippet/body or `*` for all), `labels` to scope matching, and an `action` (delete/archive/mark-as-read). Emails are normalized with `diacriticless` before matching.

**Multi-account:** The `--account` flag selects which credential/token files to use. Default account uses standard file names; named accounts use suffixed files (e.g., `gmail.credentials.work.json`).

**Config validation:** `keywords.json` is validated at load time with clear error messages including the entry index.

## Key Files

| File | Purpose |
|------|---------|
| `lib/cli.ts` | CLI entrypoint — parses flags/subcommands, wires components |
| `lib/trash-cleaner.ts` | Core rule engine — keyword validation, text normalization, matching |
| `lib/client/email-client.ts` | Base email model + abstract client/factory |
| `lib/client/gmail-client.ts` | Gmail OAuth flow — token load/save/refresh |
| `lib/client/outlook-client.ts` | Outlook MSAL auth — silent token → device-code fallback |
| `lib/client/imap-client.ts` | IMAP client (basic mail fetch/action) |
| `lib/store/file-system-config-store.ts` | Config read/write with YAML backward-compat |
| `lib/store/secure-config-store.ts` | Sensitive keys → OS keychain, others → files |
| `lib/utils/retry.ts` | Exponential backoff for 429/5xx and network errors |

## Gotchas

- **OAuth token handling differs by provider**: Gmail stores tokens as plain JSON files; Outlook uses MSAL cache plugin with persistence callbacks — don't mix the patterns
- **`keywords.json` validation is strict**: errors include the entry index — invalid entries fail the entire load, not just that rule
- **FileSystemConfigStore has YAML backward-compat**: `.json` config keys may actually load from YAML files — don't assume format from extension
- **Diacritics normalization**: emails are normalized with `diacriticless` before keyword matching — test keywords should account for this
- **`--dry-run` mode**: doesn't actually delete/archive — useful for testing but easy to forget when debugging "why didn't it work"
- **Multi-account file naming**: default account uses standard filenames; named accounts use suffixed files (e.g., `gmail.credentials.work.json`)
- **Retry logic**: `retry.ts` handles 429/5xx with exponential backoff — don't add redundant retry in calling code
- **SecureConfigStore**: sensitive creds route through OS keychain — tests must mock this or use `FileSystemConfigStore` directly

## Conventions

- **Inheritance over interfaces:** Base classes (`EmailClient`, `ConfigStore`, `ProgressReporter`, `TrashRule`) define the contract with empty/default method implementations. Subclasses override them.
- **JSDoc on all public methods:** Follow the existing JSDoc style with `@param` and `@returns` tags.
- **Test structure mirrors source:** `test/` mirrors `lib/` directory layout. Tests use Mocha + Chai (`assert` style) + Sinon for stubs/mocks.
- **Config files live in `config/`:** Sample files use `.sample` suffix. Actual credential/token files are gitignored.
- **Strict equality:** Use `===` instead of `==`; ESLint enforces this.
- **Prefix unused params with `_`:** Base class method parameters that are unused use the `_` prefix convention.

---
> Source: [hasankhan/trash-cleaner](https://github.com/hasankhan/trash-cleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
