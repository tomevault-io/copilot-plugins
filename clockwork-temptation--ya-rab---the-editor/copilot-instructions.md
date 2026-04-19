## ya-rab-the-editor

> You are a Senior System Architect and Expert DevOps Engineer specialized in scaffolding complex Next.js and TypeScript applications.


You are a Senior System Architect and Expert DevOps Engineer specialized in scaffolding complex Next.js and TypeScript applications.

## Development Conventions

- **Language:** English for code and technical comments; Arabic for business logic explanations and user-facing content.
- **Code Style:** Strict TypeScript, functional React components with Hooks.
- **Naming:** Descriptive English names; interfaces start with 'I'.

## Development Commands

**Package Manager:** pnpm (required)

```bash
# Development
pnpm dev              # Start Next.js dev server (localhost:3000)

# Building
pnpm build            # Build for production
pnpm start            # Start production server

# Testing
pnpm test             # Run tests with Vitest (watch mode)
pnpm test:run         # Run tests once
pnpm test:ui          # Run tests with Vitest UI (visual debugger)

# Code Quality
pnpm lint             # ESLint check (max-warnings=0)
pnpm lint:fix         # ESLint auto-fix
pnpm type-check       # TypeScript type checking
pnpm format           # Prettier format
pnpm format:check     # Prettier check

# Validate All
pnpm validate         # Run all checks: lint, type-check, format:check, test:run, security:check

# Other
pnpm genkit:ui        # Start Genkit UI for AI debugging
pnpm knip             # Find unused files/exports
```

## Project Overview

**Rabyana** (YA RAB - THE EDITOR) is a professional Arabic screenplay editor powered by AI, built with Next.js 16 and TypeScript. It specializes in parsing and formatting Arabic screenplay text with intelligent suggestions and automated classification of screenplay elements.

## Tech Stack

- **Framework:** Next.js 16.1.3 (App Router)
- **Language:** TypeScript 5.9 (strict mode)
- **UI:** React 19.2.3
- **Styling:** Tailwind CSS 4.1.18
- **AI/LLM:** Genkit @1.27.0 (Google AI/Gemini)
- **Testing:** Vitest 4.0.17
- **Package Manager:** pnpm 10.28.0

## Project Structure

```
src/
├── app/              # Next.js App Router pages and API routes
│   ├── api/          # API endpoints (ai/chat, gemini-classify)
│   └── editor/       # Main editor page
├── ai/               # AI-specific modules and agents
├── components/       # React UI components (dialogs, etc.)
├── config/           # Configuration files (prompts, etc.)
├── engine/           # Core screenplay parsing engine
│   ├── classifier/   # Individual element classifiers
│   ├── flow/         # Classification flow logic
│   ├── parser/       # Scene header parsing
│   ├── scoring/      # (moved to systems/scoring)
│   └── viterbi/      # Viterbi decoder for sequence optimization
├── nlp/              # Arabic NLP processing
│   ├── classifierHelpers.ts  # Pattern matching helpers
│   ├── places.ts     # Known Arabic place names
│   └── verbs.ts      # Arabic action verb patterns
├── systems/          # Advanced classification systems
│   ├── context/      # ContextAwareClassifier with caching
│   ├── memory/       # DocumentMemory for tracking characters/places
│   └── scoring/      # ScoringSystem (core scoring engine)
├── types/            # TypeScript type definitions
├── utils/            # Utility functions (text, fetchWithRetry)
└── tests/            # Test files
```

## Architecture

### Layered Classification Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                       │
│  THEditor Component → API Routes → Main Page               │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Systems Layer                           │
│  Memory System | Context System | Scoring System           │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Engine Layer                            │
│  Viterbi Decoder | Classifiers | Parser | Flow System      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    NLP Layer                               │
│  Arabic Normalizer | Verb Patterns | Places | Characters   │
└─────────────────────────────────────────────────────────────┘
```

### Screenplay Element Types

The system classifies lines into these types:

| Type                    | Description            | Example                         |
| ----------------------- | ---------------------- | ------------------------------- |
| `scene-header-top-line` | Full scene header      | "مشهد 1: المنزل - داخلي - نهار" |
| `scene-header-1`        | Scene number only      | "مشهد 1"                        |
| `scene-header-2`        | Scene location         | "بيت - داخلي"                   |
| `scene-header-3`        | Specific place         | "غرفة المعيشة"                  |
| `action`                | Action/description     | "يدخل أحمد ببطء"                |
| `character`             | Character name         | "أحمد:"                         |
| `dialogue`              | Spoken dialogue        | "مرحباً، كيف حالك؟"             |
| `parenthetical`         | Delivery note          | "(بصوت منخفض)"                  |
| `transition`            | Scene transition       | "قطع إلى:"                      |
| `basmala`               | Islamic prayer opening | "بسم الله الرحمن الرحيم"        |
| `blank`                 | Empty line             |                                 |

### Classification Methods

The [`ScreenplayClassifier`](../../src/engine/ScreenplayClassifier.ts) supports four methods:

1. **flow** - Rule-based classification using individual classifiers
2. **viterbi** - Statistical sequence optimization using Viterbi algorithm
3. **context** - Memory-enhanced classification with document context
4. **hybrid** (default) - Viterbi + context-aware review for uncertain lines

### Key Entry Points

- **Main Classifier:** [`src/engine/ScreenplayClassifier.ts`](../../src/engine/ScreenplayClassifier.ts)
- **Scoring System:** [`src/systems/scoring/ScoringSystem.ts`](../../src/systems/scoring/ScoringSystem.ts)
- **Context Classifier:** [`src/systems/context/ContextAwareClassifier.ts`](../../src/systems/context/ContextAwareClassifier.ts)
- **Document Memory:** [`src/systems/memory/DocumentMemory.ts`](../../src/systems/memory/DocumentMemory.ts)
- **Type Definitions:** [`src/types/index.ts`](../../src/types/index.ts)

### Important: Scoring System Location

The scoring system was extracted from THEditor.tsx and is now at:

- **[`src/systems/scoring/ScoringSystem.ts`](../../src/systems/scoring/ScoringSystem.ts)**

This file contains:

- `scoreAsCharacter()`, `scoreAsDialogue()`, `scoreAsAction()`, `scoreAsParenthetical()`, `scoreAsSceneHeader()`
- `classifyWithScoring()` - Main classification entry point
- `classifyBatchDetailed()` - Batch classification with context
- Doubt score calculation and smart fallback logic
- NLP helper re-exports (verbs, places, patterns)

## Testing

- Tests are located in [`src/tests/`](../../src/tests)
- Use Vitest for unit and integration tests
- Run `pnpm test:ui` for visual test debugger
- Focus testing on scoring accuracy and NLP pattern matching

## Development Workflow

1. Create feature branch from `main`
2. Write tests before implementation (TDD)
3. Implement changes in TypeScript
4. Run tests and fix issues: `pnpm test:run`
5. Run type check: `pnpm type-check`
6. Update documentation if needed
7. Create pull request

## AI/Genkit Integration

- **Genkit** framework with Google AI (Gemini) for AI-powered suggestions
- [`/api/ai/chat`](../../src/app/api/ai/chat) - AI chat endpoint for classification review
- [`/api/gemini-classify`](../../src/app/api/gemini-classify) - Direct Gemini classification
- AI review with retry logic and exponential backoff
- Use [`fetchWithRetry`](../../src/utils/fetchWithRetry.ts) for all API calls

## Arabic Text Handling

- RTL (right-to-left) text support throughout
- Arabic diacritics (تشكيل) handling
- Custom Arabic punctuation patterns
- Arabic verb patterns for action detection (verbs.ts)
- Arabic place name database (places.ts)

## Common Tasks

### Adding New Screenplay Elements

1. Add new type to [`ViterbiState`](../../src/types/index.ts) in types
2. Update [`ScreenplayClassifier`](../../src/engine/ScreenplayClassifier.ts) to handle new type
3. Add scoring function in [`ScoringSystem.ts`](../../src/systems/scoring/ScoringSystem.ts)
4. Update NLP patterns in [`src/nlp/`](../../src/nlp) if needed
5. Add tests for new element type

### Modifying Classification Rules

1. Edit scoring functions in [`src/systems/scoring/ScoringSystem.ts`](../../src/systems/scoring/ScoringSystem.ts)
2. Update related tests in [`src/tests/`](../../src/tests)
3. Run `pnpm test:run` to verify changes

## Security Notes

- Never commit API keys or secrets to version control
- Use environment variables for sensitive information
- Run `pnpm security:check` to audit dependencies

## Code Style

### TypeScript and Components

- Use TypeScript strict mode for all files
- Prefer functional components over class components
- Use React hooks for state management

### Commenting Code

- Use Arabic comments for business logic explanations
- Use English comments for technical implementation details

### Example: Scoring Function

```typescript
/**
 * scoreAsCharacter
 * @description حساب نقاط التصنيف كشخصية
 * @param rawLine السطر الأصلي
 * @param normalized السطر المُطبع
 * @param ctx سياق السطر
 * @param documentMemory ذاكرة المستند
 * @returns ClassificationScore
 */
export function scoreAsCharacter(
  rawLine: string,
  normalized: string,
  ctx: LineContext,
  documentMemory?: DocumentMemory,
): ClassificationScore {
  let score = 0;
  const reasons: string[] = [];

  // Check if character is known in document
  if (documentMemory) {
    const nameToCheck = rawLine.trim().replace(/[:：\s]+$/, "");
    const knownStatus = documentMemory.isKnownCharacter(nameToCheck);
    // ... scoring logic
  }

  return { score, confidence: "high", reasons };
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CLOCKWORK-TEMPTATION) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
