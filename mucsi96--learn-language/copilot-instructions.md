## learn-language

> !IMPORTANT: Avoid using comments which are trivial or obvious. If comment is needed means the code is not clear enough.

# AGENTS.md

!IMPORTANT: Avoid using comments which are trivial or obvious. If comment is needed means the code is not clear enough.

## General Code style
- Avoid fallback values that hide missing data - prefer crashes over silent incorrect behavior
- Prefer functional programming over procedural
- Avoid variable mutations - create new values instead of modifying existing ones
- Never use imperative loops (`for`, `while`) - use functional methods instead
- Prefer immutable data transformations over in-place modifications

## Java Code style
- Prefer using lombok
- Prefer using builder for models
- Prefer constructor injection with lombok
- Use Stream API instead of loops (`stream().map()`, `filter()`, `reduce()`, `flatMap()`, `findFirst()`, `anyMatch()`, `allMatch()`, etc.)
- Never mutate collections - use `Stream.concat()` or create new collections instead of `add()`/`addAll()`
- Use `final` for variables
- Prefer JPA projection interfaces over `Object[]` for native query results

## TypeScript Code style
- Use `const` for all variables
- Never use `push` - use spread operator (`[...arr, newItem]`) or functional methods
- Use functional array methods (`map`, `filter`, `reduce`, `flatMap`, `find`, `some`, `every`, etc.)

## Testing Code style
- Try testing from users perspective.
- Use role based selectors and selectors based on accesibility
- Avoid using `page.locator` or `page.waitForSelector` in favor of semantic selectors (`getByRole`, `getByLabel`, `getByText`, etc.)
- Avoid asserting loading state as it is not reliable
- Prefer using Playwright for end-to-end tests with TypeScript in test folder
- Don't use assertion messages
- Test every feature using E2E tests with playwright inline with existing tests

## Angular Code style
- Make sure the UI is testable using role based selectors and selectors based on accesibility
- Follow functional programming patterns from TypeScript Code style section
- Prefer using Angular Material components
- Prefer using Angular signal / resource for state management
- Prefer putting styles and templates to separate files
- Use angular resources core method when necessary
- Do  not use ::ng-deep
- Avoid using enum in TS. Prefer using string literals
- Always using new Angular templates
- Using rxjs is not allowed

## Design guidelines
- Prefer using Angular Material components
- Avoid using "Loading..." in favor of skeleton loaders

## Project Overview

This is a language learning application that uses spaced repetition to help users learn languages through digital flashcards created from PDF documents. The app combines traditional spaced repetition with AI-powered features for translations, audio generation, and contextual examples.

## Architecture

**Backend**: Spring Boot 3.5.3 with Java 21, PostgreSQL database, local file storage, Azure cloud services, OpenAI API integration, Google Gemini API integration, ElevenLabs Voices integration
**Frontend**: Angular 20 with Material UI, Azure MSAL authentication, TypeScript

### Key Technologies
- **Spaced Repetition**: Uses FSRS (Free Spaced Repetition Scheduler) algorithm via ts-fsrs library
- **AI Integration**: OpenAI GPT-4.1 for translations, ElevenLabs Voices for audio, Google Gemini API for example images
- **Cloud Services**: Local file system for PDFs/assets storage, Azure AD for authentication
- **PDF Processing**: Apache PDFBox for text extraction and document processing

## Development Commands

### Frontend (Angular)
```bash
cd client/
npm run start       # Development server
npm run build       # Production build
npm run watch       # Development build with watch mode
```

### Backend (Spring Boot)
```bash
cd server/
./mvnw spring-boot:run              # Run development server
./mvnw test                         # Run tests
./mvnw clean compile               # Clean and compile
```

### Testing
```bash
cd test/
npm test                    # Run all Playwright tests
npm run test:headed         # Run tests with browser visible
npm run test:ui             # Run tests with Playwright UI
npm run test:debug          # Run tests in debug mode
```

### Docker Development
```bash
docker-compose up           # Start all services
docker-compose up -d        # Start in background
```

## Core Entities

### Card
Primary entity representing flashcards with JSONB data containing:
- Word information (translation, gender, word type, examples)
- FSRS scheduling data (stability, difficulty, due date, state)
- Multilingual content (German base + English, Swiss German, Hungarian)
- Audio references and example images

### Source  
Represents PDF documents with metadata (name, fileName, startPage, bookmarkedPage)

### ReviewLog
Tracks review history and performance metrics for spaced repetition

## Key Services

### Backend Services
- **CardService**: Core flashcard CRUD operations, spaced repetition scheduling, statistics
- **TranslationService**: OpenAI-powered multilingual translations with context awareness
- **AudioService**: TTS audio generation and management  
- **DocumentProcessorService**: PDF text extraction and processing
- **FileStorageService**: Local file system storage for PDFs and assets

### Frontend Services
- **CardService**: Angular service for card state management and API interactions
- **SourcesService**: PDF source management and page navigation
- **BatchAudioCreationService**: Bulk audio generation with progress tracking
- **BulkCardCreationService**: Batch card creation workflows

## API Routes

### Key Endpoints
- `GET /api/sources` - List PDF sources
- `GET /api/sources/{id}/pages/{pageNumber}` - Process PDF page content
- `GET/POST/PUT/DELETE /api/cards` - Card CRUD operations
- `POST /api/translations` - AI-powered translations
- `POST /api/audio` - Generate pronunciation audio
- `GET /api/cards/due` - Spaced repetition due cards query
- `GET /api/source/{sourceId}/study-session` - Get today's existing study session (204 if none)
- `POST /api/source/{sourceId}/study-session` - Create or resume today's study session (idempotent)
- `GET /api/source/{sourceId}/study-session/current-card` - Get next card for today's session

### Frontend Routes
- `/` - Home dashboard with study overview
- `/sources` - Admin panel for managing PDF sources
- `/sources/:sourceId/page/:pageNumber` - PDF page viewer with word selection
- `/sources/:sourceId/study` - Flashcard study mode
- `/in-review-cards` - Cards pending review

## Development Patterns

### Data Storage
- Uses JSONB for flexible card data structure in PostgreSQL
- Resource-based reactive patterns in Angular for efficient data loading
- Batch operations for bulk card and audio creation

### Authentication
- Azure Active Directory integration with role-based access control
- Scopes: `readDecks`, `createDeck`
- Roles: `DeckReader`, `DeckCreator`

### AI Integration
- Context-aware translations between German, English, Swiss German, Hungarian
- Intelligent audio pronunciation generation
- Contextual example image generation

## Testing Strategy

The test suite uses Playwright for end-to-end testing covering:
- Bulk card/audio creation workflows
- Card editing and review functionality  
- Source management and page processing
- Study mode and spaced repetition logic
- User profile and authentication flows

Tests are located in the `test/tests/` directory with supporting utilities in `test/utils.ts`.

## Language Learning Features

### Spaced Repetition
- Cards have states: New, Learning, Review, Relearning
- FSRS algorithm tracks stability, difficulty, repetitions, lapses
- Due card queries optimized for study session efficiency

### Daily Study Sessions
- Study sessions are day-based: one session per source per day
- Sessions automatically resume when returning to the study page on the same day
- No session IDs in browser URLs - sessions are resolved server-side by source and date
- POST to create session is idempotent - returns existing today's session if one exists
- Sessions older than 1 day are automatically cleaned up

### Multilingual Support
- German as primary language with smart context-aware translations
- Proper grammar handling for gender, cases, word types
- Audio pronunciation support for all supported languages

### Content Creation
- Interactive PDF page viewer for word selection
- Bulk operations for efficient card creation
- AI-generated contextual examples and images

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mucsi96)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mucsi96)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
