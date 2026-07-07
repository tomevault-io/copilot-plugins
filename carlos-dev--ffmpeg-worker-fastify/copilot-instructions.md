## ffmpeg-worker-fastify

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a video processing worker service built with Fastify that downloads videos, applies transformations using FFmpeg, and uploads the results to S3/Cloudflare R2. It's designed to process TikTok-style vertical videos (1080x1920) with optional karaoke-style subtitles.

## Commands

**Development:**
```bash
npm run dev    # Start dev server with hot reload (tsx watch)
```

**Production:**
```bash
npm run build  # Compile TypeScript to dist/
npm start      # Run compiled code from dist/index.js
```

**Docker:**
```bash
docker build -t ffmpeg-worker .
docker run -p 3000:3000 --env-file .env ffmpeg-worker
```

## Architecture

### Single-File Design
The entire application lives in [index.ts](index.ts). This is intentional for simplicity and ease of deployment as a stateless worker.

### Video Processing Pipeline

The `/process-video` endpoint orchestrates a multi-stage pipeline:

1. **Download (50-60% progress)**: Streams video from URL to temp file with chunked progress reporting
2. **Subtitle Generation (60%)**: Creates ASS subtitle files with karaoke effects if `words` array provided
3. **FFmpeg Rendering (60-90%)**: Applies video filters, scaling, subtitles, audio fades
4. **Upload (90-100%)**: Sends final MP4 to S3/R2 storage

### Render Modes

**Blur Mode (default)**: Creates a blurred background that fills 1080x1920, overlays original video centered. Prevents letterboxing while maintaining aspect ratio.

**Crop Mode**: Scales to 1920px height and crops sides to fit 1080x1920. Results in zoomed, full-screen output.

### FFmpeg Settings

Optimized for fast processing with high quality (lines 252-286):
- Preset: `fast` (balanced speed/quality)
- CRF: 23 (high visual quality)
- Max bitrate: 6000k (mobile-friendly)
- H.264 high profile with yuv420p
- AAC audio at 192k
- Faststart flag for web streaming

### Progress Notifications

The `notifyBackend()` function sends webhook notifications to `BACKEND_URL/jobs/{jobId}/progress` at key milestones. These are fire-and-forget (errors don't fail the job).

### Temporary Files

All processing uses `os.tmpdir()` with UUID-based filenames. Files are always cleaned up in the `finally` block, even on errors.

## Environment Variables

Required:
- `S3_ENDPOINT` or `R2_ENDPOINT`: S3-compatible storage endpoint
- `S3_BUCKET_NAME` or `R2_BUCKET_NAME`: Target bucket name
- `S3_ACCESS_KEY_ID` or `R2_ACCESS_KEY_ID`: Storage credentials
- `S3_SECRET_ACCESS_KEY` or `R2_SECRET_ACCESS_KEY`: Storage credentials

Optional:
- `BACKEND_URL`: Webhook target for progress notifications (default: http://localhost:3000)
- `S3_REGION`: S3 region (default: 'auto' for R2)
- `PUBLIC_MEDIA_URL`: Custom CDN URL for public file access
- `PORT`: Server port (default: 3000)

Note: The `.env` file contains Supabase credentials but they're not currently used in the code.

## Key Implementation Details

### Subtitle System (lines 149-207)

The `generateSubtitleFile()` function creates ASS (Advanced SubStation Alpha) formatted subtitles with:
- Yellow text with black outline (Arial Black, 80pt)
- Karaoke timing effect (`\k` tags) that highlights words as they're spoken
- Automatic line breaking at ~20 characters to fit vertical mobile screens
- Timestamps adjusted relative to `cutStartTime` (clips may not start at 0:00)

### Video Filter Chains

**Blur mode** (line 239):
```
split → [bg: scale+crop+blur+darken] + [fg: scale sharp] → overlay center
```

**Crop mode** (line 232):
```
scale height to 1920 → crop to 1080x1920 → center crop
```

Subtitle overlay is appended to the end of either filter chain if subtitles exist.

### Error Handling

- Video download failures throw immediately
- FFmpeg errors are logged with last 1000 chars of stderr
- S3 upload errors propagate to caller
- All errors trigger error webhooks and 500 responses
- Temp files are cleaned even on failures

## Timeouts

- Fastify connection timeout: 10 minutes (600s)
- FFmpeg spawn timeout: 10 minutes (600s)
- Body size limit: 10MB (for JSON payloads, not video uploads)

These generous timeouts accommodate processing large videos or slow storage backends.

## Type Definitions

Main request body type (lines 84-91):
```typescript
interface ProcessVideoBody {
  videoUrl: string;      // URL to download source video
  startTime: number;     // Trim start (seconds)
  endTime: number;       // Trim end (seconds)
  jobId: string;         // Identifier for progress tracking
  words: Word[];         // Optional: word-level timing for subtitles
  renderStyle?: 'blur' | 'crop';  // Video scaling mode
}
```

The `Word` type (lines 78-82) provides start/end times in seconds and text content for karaoke subtitles.

---
> Source: [carlos-dev/ffmpeg-worker-fastify](https://github.com/carlos-dev/ffmpeg-worker-fastify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
