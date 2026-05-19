## bank-buckets

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

Open `index.html` directly in a modern browser — no build step, no server, no package manager. There are no tests or linters configured.

## Architecture

Plain JavaScript with no frameworks. All modules are global objects (not ES modules) loaded via `<script>` tags in `index.html` in dependency order:

```
constants.js → utils.js → storage.js → csv-parser.js → duplicate-detector.js
→ account-detector.js → bucket-suggester.js → balance-calculator.js
→ workflow-manager.js → ui.js → app.js
```

**Key globals:** `Storage`, `UI`, `WorkflowManager`, `CSVParser`, `DuplicateDetector`, `AccountDetector`, `BucketSuggester`, `BalanceCalculator`, `Utils`

## Data Flow

All state lives in `localStorage` via the `Storage` object. The app follows a 4-phase workflow managed by `WorkflowManager`:

1. **Import** — `CSVParser` parses files; `DuplicateDetector.mergeTransactions()` flags (not removes) potential duplicates; transactions saved to storage
2. **Accounts** (Phase 1) — `AccountDetector` scans transactions to suggest accounts; user confirms account type (savings vs day-to-day)
3. **Buckets** (Phase 2) — User defines virtual buckets per savings account with keyword rules
4. **Classification** (Phase 3) — Unclassified transactions are shown; user assigns them to buckets; similar-transaction modal auto-suggests bulk classification
5. **Review** (Phase 4) — `BalanceCalculator` applies classifications and starting allocations to produce final breakdown

## CSV Formats Supported

- **Qudos Bank direct export** — account number extracted from filename (`Statement_XXXXXXXX_DD.MM.YY-DD.MM.YY.csv`); headers: `Effective Date`, `Entered Date`, `Transaction Description`, `Amount`, `Balance`
- **Frollo app export** — headers include `transaction_id`, `account_number`, `amount`, `transaction_date`, etc.

PDF import is disabled (code is commented out throughout) — data parsing was unreliable.

## Duplicate Detection

`DuplicateDetector` flags duplicates rather than silently removing them. Flagged pairs are stored under `POTENTIAL_DUPLICATES` key and surfaced for user review in the Classification phase. Matching uses amount tolerance, ±1 day date window, account number, and description similarity.

## Storage Keys

All `localStorage` keys are defined in `Storage.KEYS`. The `Storage` object includes an in-memory cache layer (`_cache`) that is invalidated on every write. `Storage.clearAll()` removes all keys but `resetAllData()` in `app.js` preserves saved accounts, buckets, and starting allocations before clearing.

---
> Source: [officialpumpkin/bank-buckets](https://github.com/officialpumpkin/bank-buckets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
