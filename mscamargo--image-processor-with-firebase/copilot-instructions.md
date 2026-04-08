## image-processor-with-firebase

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a real-time image processing pipeline with a decoupled architecture:
- **Backend**: Express API + BullMQ worker for async job processing
- **Frontend**: React SPA with real-time Firebase listeners
- **Queue**: Redis-backed BullMQ for job orchestration
- **Storage**: Firebase (Firestore for metadata, Storage for images)

The worker and API are **separate processes** that must run simultaneously in development.

## Development Commands

### First-time Setup
```bash
# Backend
cd backend && npm install && cp .env.example .env
# Frontend
cd frontend && npm install && cp .env.example .env
# Start Redis
docker run -d -p 6379:6379 redis:latest
```

### Running the Application
**You must run 3 separate processes:**

```bash
# Terminal 1: API Server
cd backend && npm run dev

# Terminal 2: BullMQ Worker
cd backend && npm run worker

# Terminal 3: React Frontend
cd frontend && npm run dev
```

The worker does NOT start automatically with the API. Missing the worker means jobs will queue but never process.

### Testing
```bash
cd backend
npm test              # Watch mode
npm run test:run      # Single run
```

To run a single test file:
```bash
npx vitest tests/validation.test.ts
```

### Building
```bash
# Backend
cd backend && npm run build     # Compiles to dist/

# Frontend
cd frontend && npm run build    # Compiles to dist/
```

### Production
```bash
# Backend API
cd backend && npm start

# Worker (separate process)
cd backend && npm run worker:prod
```

## Architecture & Data Flow

### Critical Architecture Concepts

**Two-Process Backend:**
The backend is split into two independent Node processes:
1. **Express API** (`src/server.ts`) - Handles HTTP requests, validates input, creates Firestore documents, enqueues jobs
2. **BullMQ Worker** (`src/workers/imageWorker.ts`) - Processes jobs from Redis queue, performs image transformations, uploads to Firebase Storage

Both processes share:
- Firebase Admin SDK connection (service account)
- Redis connection (for BullMQ)
- Same environment variables

**Real-time Updates:**
The frontend does NOT poll. It uses Firebase's `onSnapshot` to listen to Firestore changes. When the worker updates job status/progress in Firestore, the frontend instantly re-renders.

**Job Processing Pipeline:**
1. User submits URL → Frontend validates → POST to `/api/jobs`
2. API validates with Zod → Creates Firestore doc (status: "pending") → Adds to BullMQ queue → Returns job
3. Worker picks job from queue → Updates status to "processing" → Downloads image → Processes with Sharp → Uploads to Storage → Updates Firestore with results (status: "completed")
4. Frontend listener receives Firestore update → React Context updates → UI re-renders

**Image Processing:**
All transformations happen in `src/services/imageProcessing.ts` using Sharp:
- Resize to 800x600 (fit: inside, no enlargement)
- Grayscale (same dimensions)
- Watermark (SVG overlay at bottom)
- 3 thumbnails (150x150, 300x300, 500x500) with fit: cover

All 6 processed images are uploaded to Firebase Storage **in parallel** (`Promise.all`).

**Firebase Storage Structure:**
```
processed/{jobId}/original.jpg
processed/{jobId}/resized.jpg
processed/{jobId}/grayscale.jpg
processed/{jobId}/watermarked.jpg
processed/{jobId}/thumbnail-small.jpg
processed/{jobId}/thumbnail-medium.jpg
processed/{jobId}/thumbnail-large.jpg
```

**Firestore Document Schema:**
```typescript
jobs/{jobId}: {
  id: string
  imageUrl: string
  status: 'pending' | 'processing' | 'completed' | 'failed'
  progress: number  // 0-100
  createdAt: string (ISO)
  updatedAt: string (ISO)
  error?: string
  results?: {
    original: string       // Public URL
    resized: string
    grayscale: string
    watermarked: string
    thumbnails: {
      small: string
      medium: string
      large: string
    }
  }
}
```

## Firebase Configuration

**Service Account for Backend:**
The backend requires `serviceAccountKey.json` in the `backend/` directory OR a base64-encoded version in `FIREBASE_SERVICE_ACCOUNT_BASE64` env var.

The config (`src/config/firebase.ts`) checks for base64 first (for deployment platforms like Render), then falls back to reading the JSON file.

**Security Rules:**
- **Firestore**: Public reads, server-only writes (enforced by rules, not app code)
- **Storage**: Public reads, server-only writes

If you modify Firestore/Storage from client, it will fail. Only the backend (using Admin SDK) can write.

**Frontend Firebase Config:**
Uses the web SDK (`firebase` package, not `firebase-admin`). Configured in `frontend/src/config/firebase.ts` with public API keys from `.env`.

## Adding New Features

### Adding a New Image Transformation
1. Add transformation logic to `backend/src/services/imageProcessing.ts` in the `processImage` function
2. Upload the new image in `backend/src/workers/imageWorker.ts` (add to `Promise.all`)
3. Update TypeScript types in `backend/src/types/index.ts` and `frontend/src/types/index.ts`
4. Update frontend `JobCard.tsx` to display the new image

### Adding a New API Endpoint
1. Add route in `backend/src/routes/jobs.ts` (or create new router)
2. Add validation schema in `backend/src/routes/validation.ts`
3. Add service function in appropriate `backend/src/services/` file
4. Mount router in `backend/src/server.ts` if new file

### Modifying Job Processing
Worker code is in `backend/src/workers/imageWorker.ts`. The `processImageJob` function orchestrates:
- Downloading (using `downloadService.ts`)
- Processing (using `imageProcessing.ts`)
- Uploading (using `firebaseService.ts`)
- Progress updates (via `updateJobStatus`)

BullMQ config is in `backend/src/queues/imageQueue.ts` (concurrency: 5, retries: 3).

## Common Patterns

**Error Handling in Worker:**
Always wrap in try-catch. On error, call `updateJobStatus(jobId, 'failed', 0, errorMessage)` and rethrow. BullMQ will retry up to 3 times before permanently failing.

**Adding Progress Updates:**
Call `updateJobStatus(jobId, 'processing', percentNumber)` at key milestones. Current milestones: 10%, 20%, 30%, 40%, 50%, 90%, 100%.

**Frontend Service Calls:**
- API calls: Use `src/services/api.ts` (Axios)
- Firebase listeners: Use `src/services/firebaseService.ts` (`onSnapshot`)
- Don't make direct API/Firebase calls from components

**React State:**
All job state lives in `JobsContext`. Components use the `useJobs()` hook. Don't add useState for job data in components.

## Environment Variables

**Backend (.env):**
- `REDIS_URL` - Redis connection string (required)
- `FIREBASE_PROJECT_ID`, `FIREBASE_STORAGE_BUCKET` - Firebase project details (required)
- `FIREBASE_SERVICE_ACCOUNT_PATH` - Path to service account JSON (required unless using base64)
- `FIREBASE_SERVICE_ACCOUNT_BASE64` - Base64 service account for deployment (optional)
- `MAX_IMAGE_SIZE_MB` - Max download size, default 10 (optional)
- `WATERMARK_TEXT` - Watermark text, default "Processed by CoGrader" (optional)

**Frontend (.env):**
- All `VITE_FIREBASE_*` - Firebase web config (required)
- `VITE_API_URL` - Backend URL, default http://localhost:3000 (required)

## Testing Strategy

**Unit Tests (backend/tests/):**
- Test pure functions: validation, size checks, URL validation
- Don't test Express routes directly (no mocking)
- Don't test Firebase/Redis integration (requires real services)

**Manual Testing:**
See TESTING.md for test cases. Key scenarios:
- Valid image URL (should complete with 6 images)
- 404 URL (should fail with specific error)
- Non-image URL (should fail with content-type error)
- Large image (should fail with size error)

## Troubleshooting

**Jobs stuck in "pending":**
- Check if worker is running (`npm run worker`)
- Check Redis connection (logs should show "✅ Redis connected")

**"Firebase initialization failed":**
- Verify `serviceAccountKey.json` exists in `backend/`
- Check `.env` has correct `FIREBASE_PROJECT_ID`

**Frontend not showing real-time updates:**
- Check browser console for Firebase errors
- Verify Firestore rules allow reads
- Check frontend `.env` has correct Firebase config

**Worker crashes on image processing:**
- Check Sharp installation (native module, may need rebuild)
- Verify image URL is accessible and returns valid image

## Key Files

**Backend:**
- `src/server.ts` - Express app entry point
- `src/workers/imageWorker.ts` - BullMQ worker entry point
- `src/services/imageProcessing.ts` - Sharp transformations
- `src/services/firebaseService.ts` - Firestore/Storage operations
- `src/queues/imageQueue.ts` - BullMQ queue setup
- `src/config/env.ts` - Environment validation with Zod

**Frontend:**
- `src/App.tsx` - Root component
- `src/context/JobsContext.tsx` - Global state with Firebase listener
- `src/components/JobCard.tsx` - Job display with image gallery
- `src/services/firebaseService.ts` - `subscribeToJobs` function

## Deployment Notes

Production requires 3 deployed services:
1. **API server** (Render Web Service) - runs `npm start`
2. **Worker** (Render Background Worker) - runs `npm run worker:prod`
3. **Frontend** (Vercel) - serves static build

All three need their own environment variables. API and Worker share the same backend env vars but are separate instances.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mscamargo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mscamargo)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
