## mivoa

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Testing
```bash
npm test                    # Run tests in watch mode
npm run test:run            # Run tests once (CI mode)
npm run test:coverage       # Generate coverage report
npm run test:ui             # Run tests with Vitest UI
npm run test:watch          # Run tests in watch mode
npm run test:ci             # Run full CI check: lint + typecheck + tests
npm run test:openrouter-key # Test OpenRouter API key validation
npm run test:e2e            # Run Playwright E2E tests
npm run test:e2e:ui         # Run Playwright E2E with UI
npm run test:e2e:full       # Run E2E against Firebase emulators
```

To run a single test file:
```bash
npx vitest run path/to/test.test.ts
```

To run tests matching a pattern:
```bash
npx vitest run --reporter=verbose -t "pattern"
```

### Code Quality
```bash
npm run lint                # Run ESLint
npm run lint:fix            # Fix ESLint errors automatically
npm run typecheck           # Run TypeScript type checking
```

### Build & Development
```bash
NEXT_PUBLIC_STORAGE_BACKEND=local npm run dev  # Local mode (no Firebase)
npm run dev                 # Firebase mode (requires .env.local)
npm run build               # Production build (requires NODE_ENV=production)
npm run start               # Start production server
```

## Architecture Overview

### Tech Stack
- **Framework**: Next.js 15 with App Router
- **Language**: TypeScript with strict type checking
- **Styling**: Tailwind CSS + Shadcn UI components
- **Backend**: Firebase (Auth + Firestore) — optional; replaceable via StorageBackend abstraction
- **AI**: OpenRouter API for chat and analysis features
- **Testing**: Vitest + Testing Library + Happy-DOM
- **State Management**: React Context + Custom Hooks

### Core Data Models

#### Firestore Collections Structure
```
users/{userId}/
  ├── entries/{entryId}           # Journal entries (read/write by owner)
  └── settings/{settingsId}       # User settings and API keys
```

#### Key Type Definitions

**Journal Entry** (in Firestore):
- `content`: string - main journal text
- `date`: string - date key (YYYY-MM-DD)
- `createdAt`, `updatedAt`: Timestamps
- `moods`, `themes`, `places`, `characters`: string[] - AI-extracted metadata
- `moodEmojis`, `themeEmojis`, `subjectEmoji`: emoji mappings
- `keyTakeaways`: string[] - AI-generated insights
- `conversationHistory`: ChatMessage[] - AI conversation
- `conversationSummary`: ConversationSummary - AI-generated summary
- `linkedEntryIds`: string[] - references to other entries

**Entry kinds** (`src/utils/entry-kind.ts`):
```ts
type EntryKind = 'text' | 'draft' | 'conversation';
getEntryKind(entry) // returns 'draft' | 'conversation' | 'text'
```
Prefer `getEntryKind(entry) === 'draft'` over raw `entry.isDraft` checks.

### StorageBackend Abstraction

All data access goes through the `StorageBackend` interface (`src/repositories/`), which decouples the app from Firebase:

- `storage-backend.ts`: Interface (`subscribeToAuthState`, `subscribeToEntriesByDate`, `subscribeToAllEntries`, `subscribeToSettings`, `createEntry`, `updateEntry`, `deleteEntry`, `signOut`, …)
- `firebase-storage-backend.ts`: Firebase implementation (default)
- `local-storage-backend.ts`: In-memory implementation (no Firebase required, enabled via `NEXT_PUBLIC_STORAGE_BACKEND=local`)
- `storage-provider.tsx`: React context wrapping the backend — provides `useStorage()`, `useEntriesByDate()`, `useEntry()`, `useAllEntries()`, `useSettings()`

**Primary data access pattern** — use these hooks from `storage-provider.tsx`:
- `useStorage()` → `{ user, isUserLoading, backend }` (replaces `useUser` / `useAuth`)
- `useEntriesByDate(dateKey)`, `useEntry(id)`, `useAllEntries()`, `useSettings()`

**One-shot subscription gotcha**: `LocalStorageBackend` fires callbacks synchronously on subscribe. To convert a subscription to a Promise:
```ts
const unsubscribe = backend.subscribeToAllEntries((entries) => {
  if (resolved) return;
  resolved = true;
  resolve(entries);
});
if (resolved) unsubscribe(); // handles synchronous callbacks safely
```

### Firebase Integration Patterns

**Non-Blocking Authentication** (src/firebase/non-blocking-login.ts):
- All auth operations use dedicated functions that handle offline gracefully
- Functions: `initiateAnonymousSignIn`, `initiateEmailSignUp`, `initiateEmailSignIn`, `initiateGoogleSignIn`, `initiateSignOut`
- Offline errors are caught and converted to user-friendly messages

**Non-Blocking Firestore Operations** (src/firebase/non-blocking-updates.ts):
- All Firestore writes use wrapper functions that emit permission errors without blocking UI
- Functions: `setDocumentNonBlocking`, `addDocumentNonBlocking`, `updateDocumentNonBlocking`, `deleteDocumentNonBlocking`
- Permission errors are emitted via `errorEmitter` and handled globally

**Custom Firestore Hooks** (src/firebase/firestore/):
- `useCollection`: Real-time collection subscription with loading/error states
- `useDoc`: Real-time document subscription with loading/error states
- These are low-level hooks used internally by `FirebaseStorageBackend`; prefer `useStorage()` et al. in components

### AI Services Architecture

**AI Service Layer** (src/ai/services/):
- `chat-service.ts`: Chat responses from OpenRouter
- `entry-analysis-service.ts`: Extract moods, themes, places, characters, and key takeaways
- `conversation-summary-service.ts`: Generate conversation summaries with titles and insights
- `journal-prompt-service.ts`: Generate contextual journal prompts
- `model-service.ts`: Fetch and filter available AI models from OpenRouter
- `openrouter-client.ts`: Shared OpenRouter API client with streaming support

All AI services use the OpenRouter SDK and handle streaming responses.

**AI Prompt Utilities** (src/ai/utils/):
- `prompt-builders.ts`: Builds all AI prompts — `buildDailyPromptPrompt`, `formatRecentEntriesContext`, `buildAnalysisPrompt`, `buildTemplatePromptPrompt`, `buildContextualPromptPrompt`

### Component Organization

**Component Patterns**:
- UI components in `src/components/ui/` are Shadcn components (do not modify directly)
- Feature components are co-located with related logic files (e.g., `chat-message.tsx` + `chat-message-handlers.ts`)
- Complex features are broken into multiple single-responsibility components (e.g., chat: `chat-input-form.tsx`, `chat-message.tsx`, `chat-messages-list.tsx`, `chat-typing-indicator.tsx`)
- Badge components (moods, themes, places, characters) display AI-extracted metadata with emojis

**Context Providers** (src/context/):
- `LanguageContext`: Manages i18n (English/French)
- `ModelContext`: Manages selected AI model
- `OpenRouterApiKeyContext`: Manages and validates OpenRouter API key

**Custom Hooks** (src/hooks/):
- Feature-specific hooks extract complex logic from components
- Examples: `use-entry-analysis.tsx`, `use-summary-operations.tsx`, `use-entry-linking.tsx`, `use-message-editing.tsx`

### App Handlers

`src/app/handlers/journal-handlers.ts` — business logic layer between components and `StorageBackend`:
- `saveConversationDraft` — upsert a chat draft (pass `forceCreate: true` when restoring a deleted draft)
- `deleteDraft` — remove a draft
- `saveConversationSummary` — promote a draft to a conversation entry
- `analyzeEntryContent` — trigger AI analysis

### API Routes

**Other APIs**:
- `validate-openrouter/route.ts`: Validate OpenRouter API keys (calls Firebase Identity Toolkit directly; no `api-auth.ts`)

## Code Style Standards

### Clean Code Principles
- **No comments or docstrings**: Code must be self-explanatory
- **Function naming**: Use verbs (e.g., `generateSummary`, `validateApiKey`)
- **Variable naming**: Use nouns, no abbreviations (e.g., `conversationHistory` not `convHist`)
- **Function size**: 20-40 lines max, ≤3-4 parameters (use objects for more)
- **Early returns**: Prefer early returns over nested conditionals
- **Nesting depth**: Maximum 3 levels

### File Organization
- Files should be 200-400 lines (split if larger)
- Single responsibility per file
- 1-3 exported entities per file
- Import order: Standard library → Third-party → Internal (@/*)

### TypeScript Standards
- Explicit types for public APIs
- Validate data at system boundaries (user input, external APIs)
- Prefer immutability
- Never swallow errors silently

### Firebase Abstraction Rule
**Nothing in `src/components/` or `src/hooks/` may import from `firebase/*`.**
Firebase types and instances belong only in:
- `src/firebase/` — Firebase setup and auth
- `src/repositories/firebase-storage-backend.ts` — Firebase data access

For timestamp handling in components, use `convertTimestampToDate` from `src/utils/journal-utils.ts` or duck-type as `{ toDate(): Date } | Date | string`.

### Testing Standards
- Run `npm run test:ci` before committing
- Test non-trivial logic using Arrange/Act/Assert pattern
- Tests must be fast and isolated
- Use `describe` and `it` blocks with descriptive names
- One concept per test
- Co-locate tests in `__tests__` directories

### Git Workflow
- Branch prefixes: `feature/`, `bugfix/`, `hotfix/`, `release/vX.Y.Z`
- Protected branches: `main`, `develop` (require PRs)
- Commit format: `<gitmoji> <intent>: <description>`
  - Intents: feat, fix, refactor, style, perf, fire, docs, deploy, security, wip, build
- Commit bullets should be exhaustive and list ALL changes
- PR title matches commit message format

## Environment Configuration

Copy `.env.local.example` to `.env.local`. Two modes:

- **Local mode** (default): set `NEXT_PUBLIC_STORAGE_BACKEND=local` — no Firebase required
- **Firebase mode**: set `NEXT_PUBLIC_STORAGE_BACKEND=firebase` and fill in the 8 `NEXT_PUBLIC_FIREBASE_*` variables (see `.env.local.example`)

Optional: `TEST_OPENROUTER_API_KEY` for API key validation tests.

Users configure their OpenRouter API key in the app's Settings page (stored via `StorageBackend`, not environment).

## Development Notes

- The app uses App Router (not Pages Router) - pages are in `src/app/`
- AI features require users to provide their own OpenRouter API key (BYOK model)
- `NEXT_PUBLIC_STORAGE_BACKEND=local` runs fully offline with no Firebase account
- Firebase Admin SDK is used in API routes for server-side operations (Firebase mode only)
- All dates in Firestore use Firebase Timestamp type
- Entry date keys use format: YYYY-MM-DD
- Multi-language support via `src/locales/` (en.json, fr.json)
- `recentEntries` (entries from last 7 days, max 5) flows from `use-journal-entries` → `JournalMainContent` → `JournalChat` → `useChatConversation` → system prompt in `chat-service.ts`
- Three entry types: regular entry (write mode), draft (chat in progress, not yet summarized), conversation entry (has `conversationHistory`, shows chat tab)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mariepop13)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mariepop13)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
