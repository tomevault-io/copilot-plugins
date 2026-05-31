## framedeck

> Framedeck is an AI-powered video editor. Users edit videos with natural language prompts to remove silences, add b-roll, generate subtitles, create motion designs, analyze footage, prepare voiceovers, and render finished videos.

# Framedeck - Agent Guide

Framedeck is an AI-powered video editor. Users edit videos with natural language prompts to remove silences, add b-roll, generate subtitles, create motion designs, analyze footage, prepare voiceovers, and render finished videos.

This file is the root guide for coding agents. Keep changes small, preserve the existing architecture, and prefer local project patterns over new abstractions.

## Project Overview

Framedeck is a PNPM/Turborepo monorepo with a shared `ts-rest` contract.

```text
apps/
├── frontend/          # Next.js 16, React 19, Tailwind CSS v4, shadcn, Zustand, Remotion
├── server/            # NestJS 11, AI SDK, S3, Socket.IO, ElevenLabs Scribe v2, TwelveLabs
└── media-processor/   # Rust/Axum internal service for FFmpeg audio extraction
packages/
└── api-types/         # Shared ts-rest contracts, Zod schemas, realtime constants
```

The API contract in `packages/api-types` is the source of truth for frontend hooks and server handlers. Do not hardcode backend URLs in frontend code; use the generated `api` client and env-driven base URLs.

## Required Setup

Prerequisites:

- Node.js LTS and PNPM `10.22.0`
- Rust toolchain and Cargo
- FFmpeg for media extraction and local media workflows
- App-specific `.env` files copied from each `.env.example`
- Provider credentials as needed: AWS S3/Remotion, AI Gateway or OpenAI-compatible API, ElevenLabs, TwelveLabs, Deepgram

Install dependencies from the repo root:

```bash
pnpm install
```

The root does not own a single combined env file. Configure env files per app:

- `apps/frontend/.env`
- `apps/server/.env`
- `apps/media-processor/.env`

## Development Commands

Run the full stack:

```bash
pnpm dev
```

Run without portless URL injection:

```bash
pnpm dev:direct
```

Run individual apps:

```bash
pnpm --filter frontend dev
pnpm --filter server dev
pnpm --filter media-processor dev
pnpm --filter api-types dev
```

Common checks:

```bash
pnpm typecheck
pnpm test
pnpm build
pnpm --filter frontend lint
pnpm --filter server lint
pnpm --filter media-processor lint
```

Package-specific commands:

```bash
pnpm --filter api-types build
pnpm --filter frontend exec tsc
pnpm --filter server exec tsc
pnpm --filter server test
pnpm --filter server test:e2e
pnpm --filter server test:cov
pnpm --filter media-processor test
```

Portless dev URLs:

- Frontend: `http://ai-video-editor.localhost:1355`
- Backend: `http://api-ai-video-editor.localhost:1355`
- Media processor: `http://media-ai-video-editor.localhost:1355`

Use `pnpm portless:list` to inspect active routes.

## Architecture

### ts-rest API Workflow

When adding or changing an API route:

1. Update `packages/api-types/src/index.ts` with Zod schemas and the `apiContracts` route.
2. Build shared types with `pnpm --filter api-types build`.
3. On the server, bind the route with `@TsRestHandler(apiContracts.<router>.<endpoint>)`.
4. Keep business logic in services, not controllers.
5. On the frontend, consume through `api.<router>.<endpoint>.useQuery()` or `.useMutation()`.

Request and response shape changes must start in `api-types` so frontend and server compile against the same contract.

### Upload Pipeline

1. Client calls `POST /uploads/init` to create a multipart upload and receive presigned URLs.
2. Client calls `POST /uploads/:uploadId/sign-parts` to sign chunks.
3. Browser uploads directly to S3 through `directS3Upload()` with parallel multipart upload and Transfer Acceleration.
4. Client calls `POST /uploads/:uploadId/complete`.
5. Server starts ElevenLabs Scribe v2 transcription in the background.
6. Server starts TwelveLabs video analysis in the background for video assets.
7. Results arrive over WebSocket events such as `transcriptionComplete` and `upload:videoAnalysisComplete`.

Asset statuses may include `pending-upload`, `uploading`, `in-progress`, `transcribing`, `ready`, `uploaded`, and `error`. Check existing frontend/server usage before renaming or consolidating statuses.

### Video Analysis

- TwelveLabs analysis is triggered after `POST /uploads/:uploadId/complete` for video assets.
- It runs in parallel with ElevenLabs Scribe v2 transcription and must not block upload completion.
- The pipeline is presigned S3 GET URL, TwelveLabs indexing, parallel prompt analysis, then WebSocket `upload:videoAnalysisComplete`.
- Frontend stores analysis on `asset.summary` with fields such as `macroView`, `causalLogic`, `sequentialSummary`, `socket`, and `plug`.

### Agent-Driven Editor

- `ai-gateway.service.ts` streams text and reasoning through the AI SDK and calls tools from `ToolsService.getTools()`.
- `tools.service.ts` defines Zod-validated tools named with `editorToolNames`.
- Tool calls emit WebSocket start, delta, end, and result events.
- `selectTimelineItems` dispatches through `RealtimeService`.
- `removeSilences` waits for frontend results through `registerToolResult` using tool-call-id promises.
- `WebSocketProvider.tsx` routes realtime messages by `RealtimeMessageType`.
- `editor-realtime-bridge.tsx` handles editor messages, delegates to hooks, and reports tool results back to the server.

### Media Processor

The Rust service is internal to the server. For video transcription, the server sends a presigned S3 URL to the media processor, which streams the video through FFmpeg and returns MP3 bytes for ElevenLabs Scribe v2.

Important endpoints:

- `GET /health`
- `GET /extract-audio?url=<presigned-s3-url>`

## Code Style

Core rules:

- Keep code short and simple. Avoid overengineering.
- Preserve existing module boundaries and naming.
- Prefer single quotes in TypeScript/JavaScript.
- Keep files under 300 lines when practical; extract to descriptive subfolders.
- Add comments only when they clarify non-obvious behavior. Comments must be in English.
- Use descriptive names such as `projectData` and `audioPlayerCurrentTime`, not `data` or `t`.
- Booleans use `is`, `has`, or `can` prefixes.
- Functions use action verbs such as `calculateDuration()` or `validatePermissions()`.
- Components use PascalCase with a role, such as `TranslationProgressIndicator.tsx`.
- For 3 or more function parameters, use an object parameter.

```typescript
function calculatePosition({ startTime, duration, pixelsPerSecond }: PositionParams): number;
```

Logging:

- Use `console.log` or `this.logger.log` only for short-term debugging and remove it before finishing.
- Use `console.debug` or `this.logger.debug` for long-term development logs.
- On the backend, add useful `.catch` logs so failures are debuggable.
- For important new backend functions, prefer a debug log at the beginning and a completion log at the end.

## Frontend Guidelines

- Use arrow function components: `const MyComponent: React.FC = () => { ... }`.
- Type state explicitly: `useState<string>('')`, not `useState('')`.
- Use shadcn/custom components from `apps/frontend/src/components`.
- Always use `apps/frontend/src/components/tooltip.tsx` for tooltips.
- Always use `apps/frontend/src/components/buttons/IconButton.tsx` for icon buttons.
- Use `next/image` for images.
- Upload files through `directS3Upload()` from the upload utility.
- Transcription results arrive through the `useTranscriptionListener` hook.
- Reuse existing hooks, Zustand stores, realtime bridge code, and editor utilities before adding new state paths.

Remotion lives in the frontend. When working on video compositions, rendering, captions, or Remotion dependencies, read `.cursor/rules/remotion.mdc` first and follow the local Remotion patterns. Use:

```bash
pnpm --filter frontend remotion:studio
pnpm --filter frontend remotion:deploy
```

UI-only changes outside the Remotion template folder do not require a Remotion Lambda redeploy.

## Server Guidelines

- Use the AI SDK for AI calls; check Context7 MCP for current docs before changing AI SDK usage.
- Controllers bind `ts-rest` handlers and delegate business logic to services.
- Keep upload, transcription, realtime, render, and video-analysis responsibilities in their existing modules.
- Temp files should use `temporary-files/[input|output]/[fileName]-${crypto.randomUUID()}`.
- Use `RealtimeService` for Socket.IO messages instead of emitting ad hoc events.
- For S3 multipart upload behavior, keep browser uploads direct-to-S3 and server routes focused on signing/completion.

Before implementing Supabase changes, read the most relevant guide in `docs/supabase` and follow it. If multiple guides apply, prioritize the most specific one:

- `docs/supabase/create_migrations.md`
- `docs/supabase/create_rls_policies.md`
- `docs/supabase/declarative_database_schema.md`
- `docs/supabase/postgres_sql_style_guide.md`
- `docs/supabase/create_functions.md`
- `docs/supabase/writing_supabase_edge_functions.md`
- `docs/supabase/supabase_realtime_ai_assistant_guide.md`
- `docs/supabase/bootstrap_next_js_v16_app_with_supabase_auth.md`

## Testing and Verification

Run the smallest relevant check first, then broaden based on risk.

- Shared contract changes: `pnpm --filter api-types build`
- Frontend TS changes: `pnpm --filter frontend exec tsc`
- Frontend lint-sensitive changes: `pnpm --filter frontend lint`
- Server TS changes: `pnpm --filter server exec tsc`
- Server behavior changes: `pnpm --filter server test`
- Server API flow changes: `pnpm --filter server test:e2e`
- Rust changes: `pnpm --filter media-processor test` and `pnpm --filter media-processor lint`
- Cross-package changes: `pnpm typecheck`
- Release/build confidence: `pnpm build`

Add or update focused tests when changing shared contracts, server services, upload behavior, realtime behavior, or media processing logic. If a command cannot run because of missing env or external services, report that clearly.

## Build and Deployment Notes

- `pnpm build` runs `turbo run build`.
- Turborepo build outputs include `.next/**`, `dist/**`, and `target/release/**`.
- `dev` tasks are persistent and uncached.
- Build `api-types` before server/frontend checks when contract types changed.
- Use `pnpm --filter server configure:s3-cors` when S3 CORS must be configured for browser multipart uploads.
- Use `pnpm --filter frontend remotion:deploy` after changes under the Remotion template area that need Lambda deployment.

## MCP and External Docs

Always use MCP for library documentation instead of web search when available.

- Context7: external library docs such as AI SDK, Zustand, Next.js, NestJS, and React.
- shadcn: search or install UI components.
- Figma MCP: design-to-code or code-to-design workflows.

Check current docs before changing fast-moving libraries such as AI SDK, Next.js, Remotion, Supabase, or Stripe.

## Security and Secrets

- Never commit `.env` files, provider keys, credentials, or local logs containing secrets.
- Keep browser-exposed env vars prefixed appropriately, such as `NEXT_PUBLIC_*`.
- Do not move secret-bearing operations into frontend code.
- Use presigned URLs for direct S3 access; do not expose AWS secrets to the browser.
- Preserve CORS and allowed-origin checks for frontend, WebSocket, and media routes.

## Supported Media

Audio: MP3, WAV, AAC, FLAC, OGG

Video: MP4, MOV, MKV, AVI, WebM, ProRes, including raw footage up to 50GB+

Uploads use S3 multipart upload with Transfer Acceleration for large files.

## Pull Request Guidance

- Keep PRs focused and reviewable.
- Mention contract changes, migration requirements, env changes, and external service impacts.
- Before committing, run the checks that match the changed packages.
- Do not commit generated caches, local logs, build outputs, secrets, or temporary media files.
- If hooks or CI modify files, inspect the diff before committing.

## Common Gotchas

- Update `api-types` first for API shape changes; otherwise frontend/server types drift.
- Realtime editor tools require both server tool events and frontend bridge handling.
- Upload completion should start background processing but should not wait for long-running transcription or TwelveLabs analysis.
- Portless dev scripts inject service URLs; direct mode requires env values to be correct.
- The media processor depends on FFmpeg being installed locally.
- Remotion Lambda deploys are only needed for composition/render template changes.

---
> Source: [kevinrss01/framedeck](https://github.com/kevinrss01/framedeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
