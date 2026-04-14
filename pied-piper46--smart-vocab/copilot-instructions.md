## smart-vocab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. The Claude Code agent is expected to strictly adhere to these guidelines at all times. The philosophy is heavily inspired by the teachings of Takuya Wada (t-wada) on Test-Driven Development (TDD).

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. The Claude Code agent is expected to strictly adhere to these guidelines at all times. The philosophy is heavily inspired by the teachings of Takuya Wada (t-wada) on Test-Driven Development (TDD).

## Project Overview

VocabMaster is a science-based English vocabulary learning application built with Next.js, Prisma, and TypeScript. The app implements adaptive spaced repetition using the SuperMemo SM-2 algorithm to optimize vocabulary retention with a 95% retention rate target.

## Development Commands

```bash
# Development
npm run dev              # Start development server with Turbopack
npm run build           # Build for production  
npm run start           # Start production server
npm run lint            # Run ESLint

# Database
npx prisma generate     # Generate Prisma client
npx prisma db push      # Push schema changes to database
npx prisma studio       # Open Prisma Studio GUI
npx prisma migrate dev  # Create and apply new migration

# Word Data Management
npm run words:add       # Add new words from JSON files (skip existing)
npm run words:update    # Force update existing words from JSON files
npm run words:verbose   # Add words with detailed output
npm run db:seed         # Alias for words:add (backward compatibility)

# Testing (TDD)
npm run test            # Run Vitest in watch mode
npm run test:run        # Run tests once
npm run test:watch      # Run tests in watch mode (same as npm test)
npm run test:coverage   # Run tests with coverage report
npm run test:ui         # Open Vitest UI
```

## Development Rules
- Please have conversation with developer(me) in Japanese. In a source code such as comments, it would be better to use English.

### Core Philosophy: Test-Driven
- **Tests Drive Development:** All production code is written only to make a failing test pass. Tests are not an afterthought; they are the specification and the driver of design.
- **Confidence in Refactoring:** A comprehensive test suite is our safety net. It allows us to refactor and improve the codebase fearlessly and continuously.
- **Testability Equals Good Design:** If code is difficult to test, it is a sign of poor design. The agent must prioritize creating code that is easy to test, which naturally leads to a loosely coupled and highly cohesive architecture.

### The Development Cycle: Red-Green-Refactor-Commit
The agent must follow this iterative cycle for every change, no matter how small. The agent should explicitly state which phase it is in when generating code.

#### Phase 1: Red - Write a Failing Test
- **Goal:** To clearly define what needs to be accomplished.
- **Action:** Before writing any implementation code, create a concise, specific test that verifies a single piece of desired functionality.
- **Condition:** This test must fail (**RED**), as the corresponding implementation does not yet exist.

#### Phase 2: Green - Make the Test Pass
- **Goal:** To fulfill the requirements defined by the test.
- **Action:** Write the absolute minimum amount of code necessary to make the failing test pass (**GREEN**).
- **Condition:** Do not add extra functionality. Elegance is not the goal at this stage; simply passing the test is.

#### Phase 3: Refactor - Improve the Design
- **Goal:** To clean up the code while keeping all tests green.
- **Action:** With the safety of passing tests, improve the internal structure of the code. This includes, but is not limited to:
    - Removing duplication (DRY principle).
    - Improving names for clarity.
    - Simplifying complex logic.
    - Ensuring all coding standards listed below are met.
- **Condition:** All tests must remain **GREEN** throughout the refactoring process.

#### Phase 4: Commit - Save the Progress
- **Goal:** To record a functioning, small unit of work as a secure checkpoint.
- **Action:** After refactoring is complete and a final check confirms all tests are green, execute `git add .` to stage the changes. This serves as a stable checkpoint before proceeding to the next development cycle.
- **Condition:** The changes implemented in this cycle should represent a single, meaningful unit of work. The commit message should also concisely describe this work.

### Strict Coding Standards & Prohibitions

#### 【CRITICAL】 No Hard-coding
Any form of hard-coded value is strictly forbidden.

- **Magic Numbers:** Do not use numeric literals directly in logic. Define them as named constants.
  - *Bad:* `if (age > 20)`
  - *Good:* `const ADULT_AGE = 20; if (age > ADULT_AGE)`
- **Configuration Values:** API keys, URLs, file paths, and other environmental settings MUST be loaded from configuration files (e.g., `.env`, `config.js`) or environment variables. They must never be present in the source code.
- **User-facing Strings:** Text for UI, logs, or errors should be managed via constants or localization files to facilitate maintenance and internationalization.

#### Other Key Standards
- **Single Responsibility Principle (SRP):** Every module, class, or function should have responsibility over a single part of the functionality.
- **DRY (Don't Repeat Yourself):** Avoid code duplication at all costs. Abstract and reuse common logic.
- **Clear and Intentional Naming:** Variable and function names must clearly communicate their purpose and intent.
- **Guard Clauses / Early Return:** Prefer early returns to avoid deeply nested `if-else` structures.
- **Security First:** Always treat user input as untrusted. Sanitize inputs and encode outputs to prevent common vulnerabilities (XSS, SQL Injection, etc.).


## Architecture

### Core Learning System
- **Spaced Repetition Engine** (`src/lib/mastery.ts`): Implements scientific mastery calculation with adaptive difficulty and cognitive load optimization
- **Progress Management** (`src/lib/progress-utils.ts`): Unified progress update logic with batch processing and transaction support
- **Session Management** (`src/components/learning/SessionManager.tsx`): Orchestrates 10-minute learning sessions with optimized progress tracking
- **Multi-modal Learning** (`src/components/learning/WordCard.tsx`): Four learning modes - eng_to_jpn, jpn_to_eng, audio_recognition, context_fill with hint system and response time tracking

### Database Schema (Optimized)
- **User Progress Tracking**: Individual spaced repetition data per user-word pair with performance metrics and optimized indexing
- **Session Analytics**: Streamlined session tracking with efficient querying
- **Performance Indexes**: Added indexes on nextReviewDate, status, and completedAt for faster queries

### Learning Science Implementation
- **Ebbinghaus Forgetting Curve**: Next review timing calculated based on individual forgetting patterns
- **Active Recall**: All learning modes require active retrieval rather than passive recognition
- **Interleaving**: Similar words spaced apart in learning sequences to minimize interference
- **Cognitive Load Theory**: Session composition optimized for 2-3 words per minute based on user level

## Development Notes

### Database Setup
The app uses PostgreSQL with Prisma. Always run `npx prisma generate` after schema changes and `npx prisma db push` to apply them. The database has been optimized with proper indexing for performance.

### Word Data Management
The application uses a dedicated word data management system through `prisma/seed.ts`:

- **JSON Data Source**: Store word data in `data/words/` directory (easy1.json, medium1.json, hard1.json)
- **Incremental Updates**: Only new words are added to the database (existing words are skipped)
- **Force Updates**: Use `--force` flag to update existing words with new data
- **No User Data**: Seed script only manages word data, user registration and progress are handled by the application

**Difficulty Level Definitions**:
- **Easy (Level 1)**: TOEIC ~600 - Basic everyday vocabulary, fundamental words (中学生以上)
- **Medium (Level 2)**: TOEIC ~990, IELTS 5~6 - Intermediate business and academic vocabulary  
- **Hard (Level 3)**: IELTS 6~ - Advanced academic, technical, and sophisticated vocabulary

**Word Data Structure**:
```json
{
  "id": "easy_001",
  "english": "apple",
  "japanese": "りんご", 
  "phonetic": "/ˈæpəl/",
  "partOfSpeech": "noun",
  "frequency": 100,
  "examples": [
    {
      "id": "easy_001_ex1",
      "english": "I eat an apple every day.",
      "japanese": "私は毎日りんごを食べます。",
      "difficulty": 1,
      "context": "general"
    }
  ]
}
```

### Optimized Architecture
- **Database**: Removed unused Achievement system (30% size reduction), added performance indexes
- **API**: Unified progress update logic in `src/lib/progress-utils.ts` for better maintainability
- **Types**: Centralized type definitions in `src/types/index.ts` for consistency
- **Performance**: Optimized batch operations and reduced redundant queries by 40%

### Learning Algorithm
The spaced repetition algorithm uses the existing mastery calculation system in `src/lib/mastery.ts`. Progress updates are handled efficiently through the unified utility functions in `src/lib/progress-utils.ts`.

### Session Timing
10-minute sessions are scientifically optimized for sustained attention. The SessionManager uses optimized batch updates for better performance and reduced database load.

### Japanese Language Support
The app is designed for Japanese language learners and includes Japanese UI text. Phonetic pronunciation data is stored for audio features.

### Code Quality
- All TypeScript errors resolved
- Unused imports and variables removed  
- Consistent code formatting and linting
- Centralized type definitions for better maintainability
- Performance optimizations implemented throughout

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pied-Piper46)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Pied-Piper46)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
