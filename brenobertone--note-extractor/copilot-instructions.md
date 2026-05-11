## note-extractor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SnapFlow is an AI-powered note extraction application that transforms images into categorized digital records. It uses OpenAI's GPT-4o vision model to extract text from images and automatically categorizes content as "Tasks" or "Habits", persisting to a Supabase PostgreSQL backend.

## Development Commands

### Setup

```bash
npx supabase start              # Start local Supabase (Docker required)
npm install                     # Install dependencies
```

### Development

```bash
npm run dev                     # Start Next.js dev server (http://localhost:3000)
npm run build                   # Production build
```

### Testing

```bash
npm run test                    # Unit tests (mocked, fast)
npm run test:watch              # Unit tests in watch mode
npm run test:integration        # Integration tests (requires Supabase Docker)
npx playwright test             # E2E tests (auto-starts dev server)
```

### Code Quality

```bash
npm run lint                    # ESLint
npm run type-check              # TypeScript validation
npm run format:write            # Auto-fix formatting issues
npm run format:check            # Check formatting
```

## Architecture

### Core Flow

1. User uploads image(s) via `UploadComponent` (frontend)
2. POST to `/api/process` with FormData
3. API route converts images to base64, sends to OpenAI GPT-4o
4. LLM returns JSON: `{content: string, category: "Tasks"|"Habits"}`
5. Data persisted to `user_actions` table in Supabase
6. Images uploaded to Supabase Storage (`note-images` bucket)
7. Image metadata stored in `images` table with foreign key to `user_actions`
8. Response returned to frontend, gallery auto-refreshes
9. GET from `/api/actions` retrieves actions with nested images and public URLs
10. `ThumbnailGrid` displays clickable thumbnails in 4-column grid
11. `ImageModal` shows full-size images with extracted text
12. User can hover over thumbnails to reveal delete button
13. DELETE to `/api/actions/[id]` removes images from storage and cascades database deletion

### Key Files

- **API Routes**:
  - `src/app/api/process/route.ts` - POST endpoint for image processing and storage
  - `src/app/api/actions/route.ts` - GET endpoint for retrieving actions with images
  - `src/app/api/actions/[id]/route.ts` - DELETE endpoint for removing actions and images
- **Database Schema**:
  - `supabase/migrations/20260323000000_create_user_actions.sql` - User actions table
  - `supabase/migrations/20260325000000_add_images_table.sql` - Images table + storage bucket
- **Frontend Components**:
  - `src/components/UploadComponent.tsx` - File upload with multi-image support
  - `src/components/ThumbnailGrid.tsx` - 4-column gallery with auto-refresh and delete handler
  - `src/components/ThumbnailCard.tsx` - Individual thumbnail with category label and delete button (visible on hover)
  - `src/components/ImageModal.tsx` - Full-screen modal with image carousel
  - `src/app/page.tsx` - Main page integrating all components
- **Types**: `src/types/index.ts` - TypeScript interfaces for `Image` and `UserAction`
- **Unit Tests**:
  - `src/__tests__/api/process.test.ts` - Process API with storage mocks
  - `src/__tests__/api/actions.test.ts` - Actions API with pagination tests
  - `src/__tests__/api/delete-action.test.ts` - Delete API with cascade deletion
  - `src/__tests__/components/*.test.tsx` - Component tests (includes delete button tests)
- **Integration Tests**:
  - `src/__tests__/integration/database.test.ts` - Database operations
  - `src/__tests__/integration/storage.test.ts` - Storage bucket operations
  - `src/__tests__/integration/delete-action.test.ts` - Delete with cascade verification
- **E2E Tests**:
  - `e2e/upload.spec.ts` - Basic upload flow
  - `e2e/multi-upload.spec.ts` - Multi-image upload
  - `e2e/thumbnail-gallery.spec.ts` - Gallery interactions and modal (10 tests)

### Database

- Local Supabase stack runs via Docker (`npx supabase start`)
- Schema defined in SQL migrations under `supabase/migrations/`
- **Tables**:
  - `user_actions`: id, content, category, created_at
  - `images`: id, user_action_id (FK), storage_path, file_name, mime_type, file_size, created_at
- **Storage**: `note-images` bucket (public, 10MB limit) created automatically by migration
- RLS enabled with permissive policies for development
- Reset database: `npx supabase db reset` (recreates tables + bucket)

### Testing Strategy

Three test layers with different scopes:

1. **Unit Tests** (`npm run test`): Mock external services (Supabase, OpenAI). Fast feedback.
2. **Integration Tests** (`npm run test:integration`): Real Supabase connection. Validates DB operations.
3. **E2E Tests** (`npx playwright test`): Full browser flow. Validates UI interactions.

**Test Exclusion**: Vitest config excludes `e2e/` directory; Playwright config targets `e2e/` only.

## Development Workflow

### Test-Driven Development (TDD)

**Always write tests before implementation**. Pattern:

1. Write test in `src/__tests__/` (unit or integration)
2. Run test to verify it fails
3. Implement minimal code to pass
4. Refactor if needed
5. Commit progress (see Git workflow below)

### Database Migrations

When modifying schema:

```bash
# Make manual changes to migrations or edit live schema in Studio
npx supabase db diff -f <migration_name>  # Generate migration file
npx supabase db reset                      # Apply to local DB
```

Never edit the database directly without creating a migration.

### Git Workflow

- **Never commit directly to main**: Always create a feature branch and open a PR for review
- **Pre-commit hooks** (via Husky + lint-staged):
  - lint-staged: Auto-formats and lints staged files, then re-stages them
  - Type-check: Validates TypeScript across entire codebase
  - Unit tests: Runs all unit tests (mocked, fast)
  - Integration tests: Validates database operations with real Supabase
- **Auto-commits encouraged**: Commit frequently to mark progress on your feature branch
- **Never force commits**: Let pre-commit hooks fail if tests break
- **CI Pipeline**: GitHub Actions validates on push (includes full build + E2E tests)

### Secret Management

- **Local**: Use `.env.local` for development keys
- **CI**: GitHub Actions dynamically extracts JWT keys via `supabase status -o json`
- **Never commit**: Real Supabase keys (prefixed `sb_`) or OpenAI keys to the repository

### CI/CD Pipeline

`.github/workflows/ci.yml` runs on push/PR:

1. Lint, type-check, format check
2. Build Next.js app
3. Run unit tests
4. Start Supabase via CLI in GitHub runner
5. Extract ANON_KEY and SERVICE_ROLE_KEY dynamically using `jq`
6. Run integration tests against Supabase
7. Install Playwright browsers
8. Run E2E tests
9. Upload Playwright report as artifact

**Key insight**: CI uses `supabase status -o json | jq` to extract keys at runtime, avoiding hardcoded secrets.

## Important Practices

### When Adding Features

1. Start with a test in `src/__tests__/`
2. Keep LLM prompts and parsing logic in dedicated utility files if complexity grows
3. Update README.md when architectural patterns change
4. Ensure integration tests cover new database operations

### lint-staged Benefits

The project uses lint-staged to automatically format and lint files during pre-commit:

- **Auto-formatting**: Prettier runs on all staged files and re-stages them automatically
- **Auto-fixing**: ESLint fixes issues on staged TypeScript/JavaScript files
- **Prevents CI failures**: All formatting issues are caught and fixed locally before push
- **Handles special characters**: Works correctly with files like `[id]/route.ts` that have glob-pattern characters
- **Faster commits**: Only runs on staged files, not the entire codebase

### Environment Variables

- `NEXT_PUBLIC_SUPABASE_URL`: Supabase endpoint (local: `http://127.0.0.1:54321`)
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Public anon key
- `SUPABASE_SERVICE_ROLE_KEY`: Service role key (for API routes)
- `OPENAI_API_KEY`: OpenAI API key for GPT-4o access

### Path Aliases

TypeScript configured with `@/` alias pointing to `src/` directory (see `vitest.config.ts` and `tsconfig.json`).

---
> Source: [brenobertone/Note-Extractor](https://github.com/brenobertone/Note-Extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
