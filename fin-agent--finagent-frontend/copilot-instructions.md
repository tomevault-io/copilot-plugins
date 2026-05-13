## finagent-frontend

> Keep React UI code inside `src/`. Components live in `src/components/`, shared styling in `src/App.css` and `src/index.css`, service clients such as `alpacaApi.ts` and `elevenLabsAgent.ts` sit in `src/services/`, and shared types belong in `src/types/`. Static assets are grouped under `src/assets/`, while `public/` holds files served verbatim by Vite. Automation scripts and one-off utilities go in `scripts/` (e.g. `scripts/migrate-data.ts`). Supabase configuration and database assets are maintained in `supabase/` and should mirror any backend schema changes.

# Repository Guidelines

## Project Structure & Module Organization
Keep React UI code inside `src/`. Components live in `src/components/`, shared styling in `src/App.css` and `src/index.css`, service clients such as `alpacaApi.ts` and `elevenLabsAgent.ts` sit in `src/services/`, and shared types belong in `src/types/`. Static assets are grouped under `src/assets/`, while `public/` holds files served verbatim by Vite. Automation scripts and one-off utilities go in `scripts/` (e.g. `scripts/migrate-data.ts`). Supabase configuration and database assets are maintained in `supabase/` and should mirror any backend schema changes.

## Build, Test, and Development Commands
Use `npm run dev` to launch the Vite development server with hot reload. Run `npm run build` for a production output (TypeScript project references compile first, then Vite bundles). Execute `npm run preview` to sanity-check the production build locally. Maintain lint hygiene with `npm run lint`, which applies ESLint to all source files.

## Coding Style & Naming Conventions
This project targets modern TypeScript React. Prefer functional components with hooks and keep files in PascalCase (`AccountOverview.tsx`) or kebab-case for utilities (`use-account-store.ts`). Use 2-space indentation, trailing commas where valid, and favor explicit return types for exported functions. Run ESLint before committing; configuration lives in `eslint.config.js` and extends the core React/Vite recommendations.

## Testing Guidelines
Automated tests are not yet wired into `package.json`. When contributing, add Vitest-based specs colocated beside the implementation (`Component.test.tsx`) or under `src/__tests__/`. Aim to cover business logic in `src/services/` and any reducers or hooks, and exercise UI flows with Testing Library. Document any new test commands in `package.json` and update this guide when they are available.

## Commit & Pull Request Guidelines
Follow the concise, imperative style already in the log (`fixed keys`, `changed from vin to finagent`). Scope commits narrowly and include context in the body when touching APIs or migrations. Pull requests should link to their issue, describe functional changes, list new environment variables, and attach UI screenshots or screencasts for visual updates. Request at least one peer review for feature work and confirm that lint/build checks pass before assigning reviewers.

## Environment & Security Notes
Secrets and API keys belong in a local `.env` file and should be exposed to Vite as `VITE_*` variables (`VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`). Never commit `.env` files or Supabase service keys. When editing `supabase/` migrations, run `supabase db lint` (via the Supabase CLI) before submitting changes and coordinate deploys with backend maintainers.

## Voice/UI Architecture Notes
This project uses ElevenLabs Conversational AI for voice interactions. Key architectural patterns:

### Single-Fetch Pattern
Voice webhooks (`app/api/elevenlabs/*`) return **both** `response` (for TTS) and `uiData` (for UI rendering) in a single call. This prevents voice/UI drift where the voice says one thing but UI shows different data.

### Parseable Period Suggestions
When no data is found, the system suggests available data periods. These suggestions must be parseable by `src/lib/date-parser.ts`. The `src/lib/data-availability.ts` module validates and falls back to deterministic periods if LLM suggests non-parseable dates.

### Date Parser Prefix Handling
ElevenLabs often sends time periods with prepositions like "in October", "for last week", "during September". The date parser (`src/lib/date-parser.ts`) automatically strips these prefixes (`in`, `for`, `during`) before parsing. This means:
- "in October" → parses as "October"
- "for last week" → parses as "last week"
- "during September" → parses as "September"

### Agent Prompt Updates
The ElevenLabs agent prompt lives in `prompts/finagent-neo.md`. When updating, copy to ElevenLabs dashboard. Key sections:
- Follow-up query handling (same tool with new parameters)
- "Yes" response handling (re-call tool with suggested period)
- Transaction detail queries (call same tool for breakdown)

### Voice/UI Drift Prevention

**Key Principle:** Voice is the source of truth. UI must always match what the voice agent says.

**Common drift causes and fixes:**

1. **Double Date Offset in Suggestion Follow-ups:**
   - When user says "Yes" to a data suggestion, `dateFilter` contains pre-adjusted dates
   - UI endpoints must use these dates DIRECTLY without calling `resolveDateFilter()`
   - Files: `fees-ui`, `time-trades-ui`, `account-balance-ui`

2. **Regex Detection Order:**
   - `short_interest` must be checked BEFORE `debit_interest` (both contain "interest")
   - Bulk options must be checked BEFORE "last option" (both contain "last")

3. **Fee Type Mapping:**
   - User query detection (`UnifiedAssistant.tsx` line ~871)
   - Agent response detection (`detectFeesQuery` line ~1818)
   - Both must recognize all fee types: `commission`, `credit_interest`, `debit_interest`, `locate_fee`, `short_interest`

### Short Interest Support

`short_interest` is a fee type for borrow fees on short positions:
- Maps to `LocateFee` type in FeesAndInterest database table
- Can optionally filter by symbol (e.g., "short interest for MTEN")
- Regex pattern: `/short\s+interest/i`
- Must be checked before `debit_interest` in detection order

---
> Source: [fin-agent/finagent-frontend](https://github.com/fin-agent/finagent-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
