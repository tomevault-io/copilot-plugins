## dad-social-media-agent

> The app is deployed as a single Docker container via **Coolify**.

# Docker & Coolify Deployment

The app is deployed as a single Docker container via **Coolify**.

## Docker Build Strategy

1. **Frontend Build Stage**: Node.js builds the React app into static files.
2. **Backend Runtime**: Python FastAPI serves the API and the static frontend files (SPA).
3. **Dockerfile Location**: `deploy/Dockerfile`

## Coolify Setup

- **Build Pack**: Dockerfile
- **Docker Context**: `.` (Root of the repo)
- **Dockerfile Path**: `/deploy/Dockerfile`

### Required Environment Variables

You must set these in the Coolify UI:

- `GOOGLE_API_KEY`: Google API key for Gemini 3.0 Pro (text generation) and Nano Banana Pro (image generation). Get from Google AI Studio or Vertex AI.
- `CREATOMATE_API_KEY`: For video rendering.
- `CREATOMATE_IMAGE_TEMPLATE_ID`: Template ID for image-based videos.
- `CREATOMATE_VIDEO_TEMPLATE_ID`: Template ID for video-based videos.
- `AYRSHARE_API_KEY`: For social posting.
- `ENV`: Set to `production`.
- `FRONTEND_URL`: The full URL where the app is hosted (e.g., `https://social.yourdomain.com`).

## Health Check

- Configure health check in Coolify to call `GET /health`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BethCNC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
