## leetcode-fsrs

> A Firefox extension that applies the FSRS (Free Spaced Repetition Scheduler) algorithm to LeetCode problems and concepts. Tracks both individual problems and their associated concept tags using spaced repetition scheduling.

# LeetCode FSRS - Firefox Extension

A Firefox extension that applies the FSRS (Free Spaced Repetition Scheduler) algorithm to LeetCode problems and concepts. Tracks both individual problems and their associated concept tags using spaced repetition scheduling.

## Architecture
- **Firefox Manifest V2** extension (no build tools, plain JavaScript)
- **ts-fsrs** UMD bundle vendored in `lib/` (exposes `self.FSRS` global)
- **Message-passing architecture**: background script is the single logic layer; popup and content script are UI-only clients that communicate via `browser.runtime.sendMessage()`

## File Structure
```
manifest.json          - Extension manifest (MV2)
lib/ts-fsrs.umd.js     - Vendored FSRS library (UMD, exposes self.FSRS)
src/storage.js          - All browser.storage.local CRUD (exposes self.Storage)
src/fsrs-engine.js      - FSRS wrapper, 5-button rating mapping (exposes self.FsrsEngine)
src/leetcode-api.js     - LeetCode GraphQL client (exposes self.LeetCodeAPI)
src/suggestion.js       - Weakest-concept suggestion algorithm (exposes self.Suggestion)
src/background.js       - Central message router (depends on all above)
popup/popup.html|js|css - Extension popup UI
content/content.js|css  - Floating rating bar on LeetCode problem pages
icons/                  - Extension icons (48px, 96px)
```

## Key Patterns & Conventions
- **Module pattern**: Each `src/*.js` file is an IIFE that attaches to `self` (e.g., `self.Storage = {...}`)
- **Load order matters**: Background scripts in manifest.json load sequentially: ts-fsrs → storage → fsrs-engine → leetcode-api → suggestion → background
- **Date revival**: `browser.storage.local` serializes Dates as ISO strings. `Storage.reviveCardDates()` converts them back before passing to ts-fsrs
- **No build tools**: Everything is plain JS loaded via `<script>` tags or manifest background scripts

## Rating System (5 buttons → FSRS 4 ratings)
| Button | Label           | Problem Rating | Concept Rating |
|--------|-----------------|---------------|----------------|
| 1      | Couldn't do it  | Again         | Again          |
| 2      | Right direction | Again         | Hard           |
| 3      | Barely did it   | Hard          | Hard           |
| 4      | Could do it     | Good          | Good           |
| 5      | Easily did it   | Easy          | Easy           |

Buttons 3-5 are disabled until the problem's solved status is confirmed via LeetCode GraphQL API (`status === "ac"`).

## Storage Keys
- `problems` — `{ [slug]: ProblemRecord }` with FSRS card state
- `concepts` — `{ [name]: ConceptRecord }` with FSRS card state
- `problemMeta` — `{ [slug]: ProblemMeta }` cached tags/difficulty from API
- `conceptIndex` — `{ [concept]: string[] }` reverse index: concept → problem slugs
- `reviewLogs` — `ReviewLogEntry[]` append-only history

## LeetCode API
- GraphQL endpoint: `https://leetcode.com/graphql` with `credentials: "include"`
- Query: `questionData` with `$titleSlug` variable
- `status` field: `"ac"` = solved, `"notac"` = attempted, `null` = unattempted
- Content script detects SPA navigation via URL polling (LeetCode is React SPA)

## Development
```bash
# Install ts-fsrs (only needed to update the vendored bundle)
npm install ts-fsrs
cp node_modules/ts-fsrs/dist/index.umd.js lib/ts-fsrs.umd.js

# Load in Firefox:
# 1. Go to about:debugging#/runtime/this-firefox
# 2. Click "Load Temporary Add-on"
# 3. Select manifest.json
```

## Common Gotchas
- The UMD bundle exposes `self.FSRS` (note: capital letters), accessed as `FSRS.createEmptyCard`, `FSRS.Rating.Again`, etc.
- `browser.runtime.onMessage` handler must `return true` for async responses
- Content script polling interval (2s for nav, 15s for solved check) balances responsiveness with performance
- LeetCode GraphQL requires being logged in for `status` field to work; tags work without auth

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SqrtNegativOne)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SqrtNegativOne)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
