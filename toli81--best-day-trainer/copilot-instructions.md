## best-day-trainer

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository

- **GitHub**: https://github.com/toli81/best-day-trainer
- **Branch**: `master` — Railway auto-deploys on every push
- Always `git pull origin master` before making changes.

## Commands

```bash
# Install dependencies
npm install

# Development server (localhost:3000)
npm run dev

# Production build
npm run build

# Lint
npm run eslint

# TypeScript type-check (no emit)
npx tsc --noEmit

# Drizzle ORM tools
npx drizzle-kit push
npx drizzle-kit studio
```

There are no automated tests. Manual testing is done by uploading a short (2–3 min) video and watching Railway logs for stage-by-stage pipeline output.

## Environment Variables

Required in `.env.local` (dev) or Railway environment (prod):

- `GEMINI_API_KEY` — Google Gemini API for video analysis
- `ANTHROPIC_API_KEY` — Anthropic Claude API for session notes and exercise tagging
- `R2_ACCOUNT_ID` — Cloudflare account ID
- `R2_ACCESS_KEY_ID` — R2 S3-compatible key ID
- `R2_SECRET_ACCESS_KEY` — R2 S3-compatible secret
- `R2_BUCKET_NAME` — defaults to `best-day-trainer`

## What the App Does

A full-stack tool for personal trainers to record or upload a training session video. AI automatically identifies every exercise, extracts video clips and thumbnails per exercise, generates coaching notes, and builds a searchable exercise library.

## Tech Stack

- **Next.js 16** (App Router) + React 19 + TypeScript
- **SQLite** via `better-sqlite3` + Drizzle ORM — database at `./data/bestday.db`
- **Cloudflare R2** for all video/clip/thumbnail storage (S3-compatible via `@aws-sdk/client-s3`)
- **Google Gemini 2.5 Flash** (`@google/genai`) for two-pass video analysis
- **Anthropic Claude** (`@anthropic-ai/sdk`) for session notes and exercise tagging
- **FFmpeg** (`fluent-ffmpeg` + `ffmpeg-static`) for clip extraction and thumbnails
- Deployed via **Docker on Railway** with a persistent volume at `/app/persist`

## Database

Two tables defined in `src/lib/db/schema.ts`:

- **`sessions`** — one row per uploaded video. Tracks `status` (enum: `uploading → uploaded → analyzing → segmenting → generating_notes → complete | error`) and `pipeline_stage` (checkpoint for resumability). Also stores Gemini file references (`gemini_file_uri`, `gemini_file_name`) and JSON blobs for `overview_analysis`, `details_analysis`, and `session_notes`.
- **`exercises`** — one row per exercise extracted from a session. Foreign key to `sessions` with cascade delete. Stores timestamps, R2 clip/thumbnail paths, and AI-generated metadata (name, description, muscle groups, equipment, tags, form notes, coaching cues).

Schema migrations run automatically via `ALTER TABLE ADD COLUMN` in `src/lib/db/index.ts` on startup. Sessions stuck in processing states (`analyzing`, `segmenting`, `generating_notes`) are auto-reset to `error` on startup so they can be retried.

## Upload Flow (Direct-to-R2 Multipart)

Videos never flow through Railway — the upload is client-side direct-to-R2:

1. `POST /api/upload/init` — server creates R2 multipart upload, returns presigned PUT URLs (10MB parts, 1-hour expiry).
2. Client (`src/hooks/use-upload.ts`) PUTs each part directly to R2 with retry logic.
3. `POST /api/upload/complete` — server calls R2 `CompleteMultipartUpload` with collected ETags, creates the DB session record with `videoFilePath = "r2://videos/{id}.mp4"`.
4. `POST /api/upload/cleanup` — aborts the R2 multipart upload on client-side failure.

Upload metadata between steps is held in a server-side in-memory `Map` (`src/lib/r2/upload-sessions.ts`). Both the `/record` and `/upload` pages use the same `useUpload()` hook.

## Processing Pipeline (`src/lib/processing/pipeline.ts`)

Triggered by `POST /api/sessions/[sessionId]/process`. Runs in the background (not awaited). Only **one session can process at a time** — an in-memory concurrency guard returns 409 if another session is already processing (Railway ~1GB ephemeral disk constraint).

The pipeline is **checkpoint-resumable** — each stage writes `pipeline_stage` to the DB. On retry, the pipeline resets to null and restarts from the beginning (temp files in `/tmp` don't survive Railway restarts).

Stages in order:
1. **downloaded** — download video from R2 to `/tmp`
2. **compressed** — FFmpeg compresses video for Gemini analysis (reduces upload size); records `speedFactor`
3. **uploaded_to_gemini** — uploads compressed video to Gemini Files API; persists `gemini_file_uri` and `gemini_file_name` to DB
4. **overview_complete** — Gemini overview pass: identifies all exercises with timestamps; saves JSON to `overview_analysis`
5. **details_complete** — Gemini detail pass in **batches of 3 exercises**; saves JSON to `details_analysis`
6. **clips_extracted** — FFmpeg extracts per-exercise clips and thumbnails → uploaded to R2 (`clips/{sessionId}/{id}.mp4` and `.jpg`) → exercise rows inserted into DB
7. **notes_generated** — Claude generates professional session notes
8. **complete** — Claude standardizes exercise names and generates searchability tags; Gemini file deleted from Files API

**Timestamp scaling**: Gemini analyzes a sped-up compressed video. The `speedFactor` from compression is applied when mapping Gemini's timestamps back to the original video for accurate clip extraction.

## Gemini Integration (`src/lib/gemini/`)

- `client.ts` — SDK client, `withTimeout()`, `withRetry()` (handles 429 rate limits with backoff), `uploadVideoToGemini()` with 3-attempt retry and processing-state poll loop.
- `analyze-session.ts` — `uploadAndCache()`, `analyzeSessionOverview()`, `analyzeExerciseBatch()`, `runDetailAnalysisInBatches()`, `cleanupGeminiResources()`.
- `prompts.ts` — all Gemini prompt strings.
- `schemas.ts` — Zod schemas for `ExerciseOverview` and `ExerciseDetail` that parse and validate Gemini JSON responses.

Caching is currently disabled — `gemini-2.5-flash` does not support context caching, so all calls pass the video file directly.

## Claude Integration (`src/lib/claude/`)

- `session-notes.ts` — generates markdown session notes (2-minute timeout).
- `library-manager.ts` — standardizes exercise names and generates searchability tags (2-minute timeout).

## Clips Serving (`src/app/clips/[...path]/route.ts`)

Checks for a local file first (backward compatibility for pre-R2 sessions), then redirects to a presigned R2 GET URL (1-hour expiry). Clip URLs are not permanent.

## Key Constraints

- `/tmp` does not persist across Railway restarts — the pipeline always re-downloads and re-compresses on retry regardless of `pipeline_stage`.
- Only one session can process at a time due to Railway's disk size.
- The entire recording blob sits in phone memory until upload completes — potential OOM on very long sessions.
- Processing downloads the full video (~800MB) to Railway `/tmp` — cleaned up immediately after pipeline finishes.

---
> Source: [toli81/best-day-trainer](https://github.com/toli81/best-day-trainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
