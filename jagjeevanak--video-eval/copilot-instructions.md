## video-eval

> **VidEval** is a browser-based (client-side, local-first) Next.js 15 app for evaluating videos against custom rubrics using AI providers. Teachers upload a rubric file, select a Google Drive folder with videos, choose an AI provider + API key, and the app evaluates each video and stores results in Google Sheets.

# VidEval — Project Guide for AI Coding Agents

## What This Project Is

**VidEval** is a browser-based (client-side, local-first) Next.js 15 app for evaluating videos against custom rubrics using AI providers. Teachers upload a rubric file, select a Google Drive folder with videos, choose an AI provider + API key, and the app evaluates each video and stores results in Google Sheets.

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript
- **State**: Zustand (persisted to localStorage)
- **UI**: shadcn/ui + Tailwind CSS
- **AI Providers**: OpenAI, Claude, Gemini, OpenRouter, Groq (all called directly from browser via `fetch`)
- **Storage**: Google Sheets (results), Google Drive (videos), localStorage (app state)
- **Auth**: Google OAuth (GSI)

## Key Directories

```
src/
  app/                          # Next.js App Router
    page.tsx                    # Public homepage
    dashboard/page.tsx          # Authenticated dashboard
    create/page.tsx             # Create evaluation room
    room/[id]/page.tsx          # Room detail (video evaluation)
    settings/page.tsx           # Settings
    api/rubrics/parse/route.ts  # Server-side rubric file parsing
    api/rubrics/summarize/route.ts  # Server-side rubric summarization
  screens/
    RoomDetail.tsx              # Main evaluation orchestrator
    CreateRoom.tsx              # Room creation form
    Dashboard.tsx               # Dashboard screen
    GoogleAuthScreen.tsx        # Google OAuth
  lib/
    aiEvaluator.ts              # Core AI evaluation + transcription
    googleApi.ts                # Google OAuth, Drive, Sheets API
    rubricParser.ts             # Rubric extraction + prompt generation
    rubricSummarizer.ts         # AI rubric summarization
    videoProcessor.ts           # Video clip splitting + screenshot extraction
  stores/
    useAppStore.ts              # Zustand store (rooms, auth, roomVideos)
  types/
    index.ts                    # TypeScript types
```

## Evaluation Flow

1. **Auth** — User signs in with Google OAuth → access token stored in Zustand
2. **Create Room** — User picks Drive folder, uploads rubric (CSV/PDF/DOC/DOCX), selects AI provider + enters API key
3. **Evaluate** — For each video:
   - Split into 30-second clips (`videoProcessor.ts`)
   - Extract screenshot at midpoint of each clip
   - Get full transcript, split proportionally by time per clip
   - Evaluate each clip with AI using transcript + screenshot (`aiEvaluator.ts`)
   - Average all clip scores → final video score
   - Append to Google Sheet (`googleApi.ts`)
4. **View Results** — Table shows scores per rubric, descriptions, Google Sheet link

## State Management (Zustand)

Store: `useAppStore`
- `auth` — `{isAuthenticated, accessToken, userEmail, userName, userPhoto}`
- `rooms` — array of `Room` objects (persisted to localStorage)
- `roomVideos` — `Record<roomId, VideoFile[]>` (NOT persisted)

Key: `video-eval-store` in localStorage. Partializes `auth` and `rooms` only.

## AI Providers & Models

| Provider | Default Model | Vision Support |
|----------|--------------|----------------|
| openai | gpt-4o | Yes (image_url) |
| claude | claude-sonnet-4-20250514 | Yes (image content part) |
| gemini | gemini-2.5-flash | Yes (inlineData) |
| openrouter | google/gemini-2.5-flash-preview | Yes (image_url) |
| groq | meta-llama/llama-4-scout-17b-16e-instruct | Yes (image_url) |

All API calls are made **directly from the browser**. API keys are user-entered at room creation time.

## Google Sheet Storage

- Spreadsheet created in the same Drive folder as the source videos
- Headers: `["Sr.", "Name", "Clips Evaluated", ...rubricNames, "Description"]`
- Each evaluation appends one row with averaged scores across all clips
- Spreadsheet ID stored on the `Room` object

## Important Conventions

- All AI calls use `fetch` (no SDKs)
- JSON responses from AI are parsed via regex `\{[\s\S]*\}`
- Screenshots are JPEG at max 640px width (base64 encoded)
- Clip duration: 30 seconds
- Scores are averaged and rounded to 2 decimal places
- No backend database — everything is client-side
- Google OAuth uses Google Identity Services (GSI) with token client
- Components use shadcn/ui pattern with `cn()` utility for className merging

## Environment Variables

- `NEXT_PUBLIC_GOOGLE_CLIENT_ID` — Google OAuth client ID
- `NEXT_PUBLIC_GOOGLE_PICKER_API_KEY` — Google Picker API key
- `NEXT_PUBLIC_GOOGLE_APP_ID` — (optional) Google Cloud project number

AI API keys are NOT env vars — entered by user at runtime.

## Implementation Docs

See `implementation_docs/` for detailed implementation notes:
- `clip-based-evaluation.md` — Clip evaluation architecture and flow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JagjeevanAK) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
