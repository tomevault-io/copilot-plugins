## macos-mcp

> Local MCP server bridging Claude to six macOS apps (Reminders, Calendar, Notes, Mail, Messages, Contacts). The user's data is the truth: if Mail.app shows 7 messages, this server returns 7. Any discrepancy with the native app is a bug.

# CLAUDE.md

Local MCP server bridging Claude to six macOS apps (Reminders, Calendar, Notes, Mail, Messages, Contacts). The user's data is the truth: if Mail.app shows 7 messages, this server returns 7. Any discrepancy with the native app is a bug.

## ⛔ MERGING TO `main` PUBLISHES A PUBLIC NPM PACKAGE

This repo is **publish-on-merge**. `.github/workflows/ci.yml` runs `semantic-release`
on every push to `main`, which publishes to npm as `mcp-macos` and cuts a GitHub
release. That is a public, outward-facing act and it is not cleanly reversible — npm
restricts unpublish after 72 hours.

**So merging a PR here is not the same class of action as merging a PR in any other
repo on this machine.** It is a release.

- **`fix:` / `feat:` / `perf:` on `main` → a real npm publish.**
- **`docs:` / `chore:` / `style:` / `refactor:` / `test:` on `main` → CI runs, nothing publishes.**
- ⚠️ **Exception: a `BREAKING CHANGE:` footer publishes a MAJOR from any commit type**, `docs:`
  included, and a `revert:` publishes a patch. Verified at source in the installed
  `@semantic-release/commit-analyzer@13.0.1` `lib/default-release-rules.js` — that file is the
  authority, not this list.
- `pnpm release:preview` dry-runs it and prints the next version before you commit to anything.

**Rules for any agent, and especially for an unattended one:**

1. **Never merge a `fix:`/`feat:`/`perf:` PR here without Kyle saying so in that session.**
   A general instruction to "get everything back on main" or "clean up the branches"
   is NOT that authorization — those sweeps are written for repos where a merge is
   recoverable, and here it is a publish.
2. **A PR held for release timing is left as a DRAFT on purpose.** Do not mark it ready
   to clear a queue. A deliberate hold is marked on four surfaces — draft flag, a
   `[HELD — DO NOT MERGE: ...]` title prefix, a blockquoted notice as the first block of
   the body, and a `do-not-merge:publishes-npm` label. The title prefix matters most: a
   sweep's inventory pass reads titles, not bodies. Set all four when you park one.
3. Doc-only changes are free — commit them normally.

⚠️ **Do not "solve" this by deleting or renaming the local clone.** `~/.local/bin/EventKitCLI`
is a symlink into `bin/` of this working copy, so removing it silently breaks every
Reminders and Calendar call on this machine — a failure that has already recurred twice
(2026-07-21, 2026-08-19). The guard belongs on the PR and in this file, not in the
filesystem.

## Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build TypeScript + Swift binary (required before running)
pnpm test             # Run all tests
pnpm lint             # Lint and format with Biome
pnpm dev              # Run from source via tsx (stdio only, no build needed)
pnpm test:e2e         # Build + run functional E2E tests (node:test, real OS calls)
pnpm test:e2e:all     # Build + run ALL per-tool E2E suites (serial, no JXA contention)
pnpm release:preview  # Dry-run semantic-release (shows next version + changelog)
```

`bin/run.cjs` runs compiled `dist/index.js` — requires `pnpm build`. Use `pnpm dev` for source-level iteration (stdio only).

## Known Gotchas

- **Calendar**: Recurring event deletion only removes single occurrence (`.thisEvent` span)
- **Notes folders**: No rename/delete via JXA (Apple API limitation)
- **Messages**: No delete/edit via JXA or SQLite (Apple API limitation)
- **Messages reads**: JXA broken on Sonoma+ — all reads use SQLite at `~/Library/Messages/chat.db`
- **Mail create**: Creates draft only — user must click Send in Mail.app
- **Mail reads**: Use SQLite (`~/Library/Mail/V10/MailData/Envelope Index`), JXA for writes only. See ADR-001
- **Gmail labels**: Messages live in `[Gmail]/All Mail`. Folder membership is in the `labels` join table. Both `listInboxMessages()` and `listMailboxMessages()` check this. See ADR-001 addendum
- **JXA rule**: Always use `whose()` predicates for search, never JS iteration over collections
- **Notes title**: Apple re-derives `n.name()` from body on every `n.body =` — update/append scripts re-set name after body assignment
- **EventKit date range**: `predicateForEvents` cannot span >4 years. Both `findEventById` and `findEvents` default to ±2 years
- **Contact enrichment at scale**: Per-handle JXA lookups don't scale beyond ~10 participants

## Architecture

Three bridges to Apple apps:

- **EventKit (Swift binary)** — Reminders, Calendar
- **JXA** — Notes, Mail writes, Contacts, Messages send
- **SQLite** — Messages reads (`~/Library/Messages/chat.db`), Mail reads (`~/Library/Mail/V10/MailData/Envelope Index`)

### Key Files

```
src/
├── config/              # Zod schema + loadConfig() (auto-injects name/version from package.json)
├── server/
│   └── server.ts        # MCP server factory
├── tools/
│   ├── definitions.ts   # MCP tool schemas (dependentSchemas for validation)
│   ├── index.ts         # Tool routing
│   └── handlers/        # Domain handlers (reminderHandlers.ts, notesHandlers.ts, etc.)
├── utils/
│   ├── cliExecutor.ts          # Swift binary execution + permission retry
│   ├── contactResolver.ts      # Cross-tool contact enrichment (name ↔ phone/email)
│   ├── jxaExecutor.ts          # JXA/AppleScript execution + retry logic
│   ├── logging.ts              # Structured error logging (tool failures → stderr)
│   ├── sqliteContactReader.ts  # Contact enrichment cache via AddressBook SQLite (ADR-002)
│   ├── sqliteMailReader.ts     # Mail SQLite reader (ADR-001)
│   ├── sqliteMessageReader.ts  # Messages SQLite reader
│   └── preflight.ts            # Startup --check validation
└── validation/
    └── schemas.ts       # Zod schemas
```

### Data Flow

- **EventKit**: Handler → Repository → `executeCli()` → Swift binary → JSON
- **JXA**: Handler → `buildScript()` → `executeJxaWithRetry()` → osascript → JSON

### Key Patterns

- **Zod validation**: All inputs validated via schemas in `validation/schemas.ts`
- **Error handling**: Use `handleAsyncOperation()` from `errorHandling.ts`. Permission errors include System Settings deep-link URLs.
- **JXA safety**: Always use `sanitizeForJxa()` before interpolating user input
- **Date formats**: `YYYY-MM-DD HH:mm:ss` for local time, ISO 8601 for UTC

### Contact Enrichment

Cross-tool layer resolves phone numbers and emails to contact names (Messages, Mail, Calendar).

- **Handle→name**: SQLite bulk cache from AddressBook DB (<50ms for 1100+ entries). See ADR-002
- **Name→handles**: Targeted JXA `whose()` search — O(log n)
- **Startup**: `warmCache()` called fire-and-forget, prevents cold cache timeouts
- **Safety**: All enrichment paths protected by `withTimeout(5000ms)`
- **Toggle**: `enrichContacts` param (default: true)

## Testing

- **Unit**: Jest with ts-jest ESM. Coverage thresholds in `jest.config.mjs` (95%/80%/95%/95%). If coverage drops, write tests for uncovered branches.
- **E2E**: `pnpm test:e2e` — node:test suite, real MCP client, real OS calls. Tests use `[E2E-TEST]` prefix for cleanup.
- **Mocks**: `src/utils/__mocks__/cliExecutor.ts`, `src/tools/jxaHandlers.test.ts`

## Development Workflow

This is a public repository.

- **Features and fixes** → branch + PR. Issue first, link it (`Closes #XX`)
- **Chores, docs, config** → direct to `main`. No PR ceremony for trivial changes
- Pre-push hook (`scripts/hooks/pre-push`) runs tests + build automatically

## Releasing

Before any release, run `pnpm release:preview` and verify the version bump matches intent. `fix:` → patch, `feat:` → minor, `BREAKING CHANGE:` → major.

## Distribution

- **npm**: Published as `mcp-macos`
- **CI**: GitHub Actions — test + lint + release (#86)

## Competitive Landscape

- `mattt/iMCP` (1418 stars) is the long-term threat — Swift `.app` + menu-bar UX, covers Messages/Contacts/Calendar/Reminders but **not Mail or Notes**. Different product category (ships as `.app`, not npm). If iMCP adds Mail+Notes, the wedge closes.
- `supermemoryai/apple-mcp` (3087 stars, JXA-based) was archived January 2026. Still pulls 1.1k npm dl/mo on inertia.
- Differentiators to preserve: SQLite reads for Mail (`Envelope Index`) and Messages on Sonoma+, cross-tool contact enrichment via SQLite AddressBook cache. Nobody else combines these.
- Posture: quietly available. No registry submission, no `awesome-mcp-servers` PR, no announcement.

## Scope Discipline

- If a subsystem has no external callers AND Kyle doesn't remember what it does, default to **cut**, not preserve. Two no's = dead weight regardless of LOC.
- Post-pivot, deletion is highest-leverage work. INTENT.md framing changed (phone-from-coffee-shop → local-only on Mac Mini); the right response was cutting HTTP transport + opinionated prompts subsystem, not maintaining them.

## Commits

Use `/commit` skill.

---
> Source: [krmj22/macos-mcp](https://github.com/krmj22/macos-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
