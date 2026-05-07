## context

> This project is an AI-powered Podcast Assistant built with:


# AI Podcast Assistant — Project Context (MDC)

## Overview
This project is an AI-powered Podcast Assistant built with:
- **Next.js 15**
- **Clerk** (Auth + Billing)
- **Convex** (Database + Realtime)
- **Vercel Blob** (File storage for audio/video + outputs)
- **Inngest** (Background job orchestration)
- **OpenAI Whisper & GPT** (Transcription + content generation)
- (Optional) **Cloudflare R2** for long-term storage of heavy artifacts

The system processes a podcast **audio or video** file and generates:
- Summary
- Key timestamps (clip opportunities)
- Multi-platform social posts
- Captions (SRT + raw)
- Title suggestions
- Hashtags
- Full transcript

No video clipping.  
No thumbnail generation.

---

## User Flow

1. **Upload**
   - User selects a podcast audio/video file on the client.
   - Client submits a `FormData` POST to a Next.js route: `POST /api/upload`.

2. **Upload Route (`/api/upload`)**
   - Validates user via **Clerk**.
   - Reads the `file` from `FormData`.
   - Uploads the file to **Vercel Blob** using `put()`:
     - Key pattern: `${userId}/${uuid()}.${ext}`
     - `access: "public"` (or private if we later use signed URLs).
   - Creates a `project` record in **Convex** (or at least has the information to later link results).
   - Triggers an **Inngest** event: `podcast.uploaded` with:
     - `fileUrl`
     - `userId`
     - `type` = `"audio"` or `"video"`
     - (Optional) `projectId`
   - Returns JSON to client:
     - `{ url: fileUrl, projectId? }`

3. **Inngest Workflow**
   - Runs the processing pipeline in **two phases**:
     - **Linear phase**:
       - Extract audio (if video)
       - Transcribe audio
     - **Parallel phase** (fan-out based on transcript):
       - Generate key moments (timestamps)
       - Generate podcast summary
       - Generate platform-optimized social posts
       - Generate captions (SRT + raw text)
       - Generate title suggestions
       - Generate hashtags
   - Joins all results and:
     - Stores structured data in **Convex**
     - Stores heavy artifacts (.srt, transcript JSON) in Blob/R2
   - Updates project status to `completed`.

4. **Dashboard**
   - Client subscribes to Convex for project updates.
   - User sees:
     - Summary
     - Key timestamps
     - Captions (SRT download)
     - Social posts
     - Titles
     - Hashtags
     - Full transcript

---

## Inngest Pipeline Steps (With Parallel Jobs)

All steps should be implemented as `step.run`.  
The **linear** steps run sequentially; the **content generation** runs in parallel after transcription using `Promise.all`.

**Linear phase:**
1. `extractAudioIfVideo`
2. `transcribeAudio`

**Parallel phase (fan-out):**
3. `generateKeyMoments`  
4. `generatePodcastSummary`  
5. `generateCaptions` (.srt + raw transcript)  
6. `generateSocialPosts` (Twitter, LinkedIn, IG Caption, TikTok, YouTube Description, Facebook)  
7. `generateTitles` (short/long/podcast/SEO)  
8. `generateHashtags` (per platform)

**Join phase:**
9. `saveResultsToConvex`

After `transcribeAudio`, the workflow should call `Promise.all([...step.run(...)])` to execute the content-generation steps in parallel, then join their results and persist.

---

## Convex Data Model (Project Document)

Each upload corresponds to a single **project** document. Parallel Inngest jobs populate its fields over time.

```ts
{
  _id: Id<"projects">,
  userId: string,
  inputUrl: string, // Vercel Blob URL

  status: "uploaded" | "processing" | "completed" | "failed",

  jobStatus?: {
    transcription?: "pending" | "running" | "completed" | "failed",
    contentGeneration?: "pending" | "running" | "completed" | "failed"
  },

  transcript: any, // full transcript JSON

  keyMoments: { time: string; text: string }[],

  summary: {
    full: string,
    bullets: string[],
    insights: string[],
  },

  captions: {
    srtUrl: string,   // Blob URL to .srt
    rawText: string,  // plain text for captions
  },

  socialPosts: {
    twitter: string,
    linkedin: string,
    instagram: string,
    youtube: string,
    tiktok: string,
    facebook: string
  },

  titles: {
    youtubeShort: string[],
    youtubeLong: string[],
    podcastTitles: string[],
    seoKeywords: string[]
  },

  hashtags: {
    youtube: string[],
    instagram: string[],
    tiktok: string[],
    linkedin: string[],
    twitter: string[]
  },

  createdAt: number
}


⸻

Storage Layout (Vercel Blob / R2)

Use Blob for simplicity; optionally mirror or migrate to R2 for cost.

Example key layout:

blob://uploads/{userId}/{uuid}/input.(mp4|mp3|wav)
blob://transcripts/{projectId}/transcript.json
blob://captions/{projectId}/captions.srt

Convex stores only URLs, never raw file buffers.

⸻

Authentication & Billing
	•	Clerk handles user accounts, sessions, orgs.
	•	Upload route (POST /api/upload) must require an authenticated user.
	•	Clerk Billing (Stripe) restricts:
	•	Max duration per file
	•	Number of projects per month
	•	Concurrent processing limits
	•	Access to certain features (e.g., export, bulk processing)

⸻

UI Requirements

Upload Page
	•	File input or drag & drop.
	•	On submit:
	•	POST to /api/upload with FormData.
	•	Show “Uploading…” → “Processing…”.
	•	Store returned projectId (if present) to link to dashboard.

Project Dashboard

Per project, show:
	•	Status (uploaded / processing / completed)
	•	Summary
	•	Key timestamps
	•	Captions (download SRT)
	•	Social posts (per platform)
	•	Title suggestions
	•	Hashtags
	•	Full transcript

The UI should use Convex subscriptions / queries to update in realtime as Inngest fills the document.

⸻

Out of Scope
	•	No video clipping
	•	No thumbnail generation
	•	No visual editing tools
	•	No live streaming

⸻

Goal

Create the cleanest, most reliable AI workflow SaaS that turns any podcast audio/video into:
Summaries, timestamps, captions, social posts, titles, hashtags & transcript, using Vercel Blob + Inngest (parallel jobs) + Convex + Clerk.

---
> Source: [sonnysangha/ai-podcast-saas-nextjs-16-inngest-clerk-ai-agent-coderabbit](https://github.com/sonnysangha/ai-podcast-saas-nextjs-16-inngest-clerk-ai-agent-coderabbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
