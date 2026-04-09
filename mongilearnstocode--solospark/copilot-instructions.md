## solospark

> * Create `/components/post/PostEditor.tsx`


# Phase 3 Implementation Checklist

## 1. Post Editor UI
* Create `/components/post/PostEditor.tsx`
   * Add text input for caption content
   * Add media upload (images/videos)
   * Add platform toggle buttons (Instagram, X, LinkedIn)
   * Add per-platform customization inputs
   * Add "Suggest Caption" button (see AI Integration)
* Use **React Hook Form** for form state
* Use **Zod** for validation schemas

## 2. AI Caption Suggestions
* Set up `/lib/aiClient.ts`
   * Connect to OpenAI API (GPT-4o-mini)
   * Sanitize user input before sending
   * Handle errors and return fallback response if OpenAI fails
* Add button in PostEditor to fetch caption suggestions
* Display suggestions in UI (e.g., dropdown, modal, or inline)
* Implement rate limiting using **Upstash Redis**
   * Cap: 10 requests per minute per user

## 3. Platform-Specific Tweaks
* Update PostEditor to:
   * Show platform-specific limits and rules (e.g., X character count)
   * Add inline validation for per-platform constraints
   * Preview per-platform output

## 4. Scheduling Interface
* Add date/time picker in PostEditor
   * Use react-datepicker or similar component
* Add "AI Suggest Time" button (mocked)
   * Return fixed smart-looking times
* On submit, queue post via BullMQ

## 5. Visual Calendar
* Build `/components/calendar/CalendarView.tsx` using **FullCalendar**
   * Display posts from database
   * Enable drag-and-drop rescheduling
   * Allow edit/delete actions on click
* Integrate calendar with backend (`getScheduledPosts`, `updatePostSchedule`, etc.)

## 6. Backend Integration (tRPC)
* Create tRPC procedures in `/lib/trpc.ts`
   * `createPost`
   * `getScheduledPosts`
   * `updatePostSchedule`
   * `deletePost`
* Ensure each procedure:
   * Checks Clerk authentication
   * Uses correct `userId` scoping
   * Validates inputs using Zod
   * Handles errors gracefully

## 7. Job Queue Integration
* In `/lib/queue.ts`
   * Define job queue schema and job metadata structure
   * Add `schedulePost` job to queue with scheduled time
* In `/lib/worker.ts`
   * Handle scheduled jobs
   * Log post metadata when job executes
   * Write to `JobLog` in Supabase

## 8. UI & UX Polish
* Ensure responsiveness (PostEditor and Calendar) for mobile
* Add feedback indicators:
   * Success toast on post schedule
   * Error banners on failed submission
   * Loading states for AI and API calls

## 9. Testing
* Write unit tests for:
   * `aiClient.ts`
   * tRPC procedures
* Write integration tests for:
   * Post creation and scheduling flow
   * Calendar fetch/update actions
* Validate rate limit logic works in test/dev mode

## 10. Final Review & Deployment
* CI passes with new tests and lint checks
* Phase 3 deploys cleanly to Vercel
* Manual QA: end-to-end post creation → calendar schedule → job log

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MongiLearnsToCode)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MongiLearnsToCode)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
