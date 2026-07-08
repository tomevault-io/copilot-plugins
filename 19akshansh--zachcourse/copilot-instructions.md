## zachcourse

> This document describes every AI agent in ZachCourse, how it works, what it uses, and where the code lives.

# ZachCourse — Agent Architecture

This document describes every AI agent in ZachCourse, how it works, what it uses, and where the code lives.

---

## Overview

ZachCourse is built around **seven specialized agents**, each responsible for a distinct part of the learning experience. They share a common infrastructure (Vercel AI SDK v7, Gemini via `@ai-sdk/google`, Zod schemas, NeonDB) but operate independently with no shared state at runtime.

```
User Request
     │
     ▼
Express Server (server.ts)
     │
     ├── POST /api/generate-roadmap   → Roadmap Agent
     ├── POST /api/generate-visual-roadmap → Visual Roadmap Agent
     ├── POST /api/generate-lesson    → Lesson Agent
     ├── POST /api/generate-quiz      → Quiz Agent
     ├── POST /api/mentor-chat        → Mentor Agent  ← agentic loop
     └── tRPC /api/trpc               → Progress Agent (DB reads/writes)
```

All routes protected by `requireAuth` middleware (Better Auth session check) and `aiRateLimit` (20 req/min per IP via `express-rate-limit`).

---

## Course Personalization Engine (Tone & Background Context)

ZachCourse implements a centralized personalization engine that dynamically alters the voice, formatting, and complexity of every single agent's response based on the learner's preferences:
1. **Personalization Fields:**
   - `experienceLevel`:🌱 Beginner, 🔥 Intermediate, 🚀 Advanced.
   - `backgroundContext`: A 500-character free-text field highlighting past programming languages, favorite learning styles, and goals.
   - `tone`: A 4-way style guideline configuration (Professional, Friendly, Gen Z, ELI5) stored in `src/lib/tone-options.ts`.
2. **Multi-Agent Threading:**
   - **Roadmap / Visual Roadmap Agents:** Adapts lesson topics, graph node counts, sequencing, and pacing parameters.
   - **Lesson Agent:** Adapts the explanation style, vocabulary complexity, and introductory pacing.
   - **Quiz Agent:** Shapes question scenarios, options, and explanations to mirror the chosen style.
   - **Mentor Agent:** Feeds system guidelines directly into conversational system prompts for adaptive tutee chats.
   - **Project Generation (generateProject):** Automatically extracts the course's `tone` and `backgroundContext` to customize the description, steps, and success criteria for module-end projects.

---

## Agent 1 — Roadmap Agent

**File:** `server.ts` → `POST /api/generate-roadmap`

**Purpose:** Takes a topic, URL, or raw syllabus text and generates a structured week-by-week learning roadmap.

**Technique:** `generateObject` with a strict Zod schema — the model is forced to return validated JSON matching the schema exactly. No free-text parsing required.

**Input:**
```typescript
{
  topic: string,          // max 500 chars
  sourceUrl?: string,     // optional reference URL
  textContent?: string,   // optional pasted syllabus, capped at 3000 chars in prompt
  experienceLevel: "beginner" | "intermediate" | "advanced",
  backgroundContext?: string, // optional learner background and preferences
  weeklyHours: number
}
```

**Output schema (Zod):**
```typescript
z.object({
  title: z.string(),
  description: z.string(),
  difficulty: z.string(),
  totalDuration: z.string().optional(),
  prerequisites: z.array(z.string()),
  modules: z.array(z.object({
    id: z.string(),
    title: z.string(),
    description: z.string(),
    lessons: z.array(z.object({
      id: z.string(),
      title: z.string(),
      duration: z.string(),
      concepts: z.array(z.string()),
      difficulty: z.string().optional(),
      type: z.string().optional(),
      description: z.string().optional(),
    }))
  }))
})
```

**Prompt rules:**
- 3–5 modules, each with 3–5 lessons
- Progressive difficulty
- Last lesson must be a hands-on project
- First lesson completable in under 30 minutes
- Depth adapts to `experienceLevel` ( beginner/intermediate/advanced tailoring)
- Course pacing and module density scale with `weeklyHours` commitment (ranging from casual to intensive)

**Fallback:** `getLocalFallbackRoadmap()` — returns a hardcoded template if all Gemini models fail.

**Model:** `callAI()` helper — tries `gemini-2.5-flash` → `gemini-2.0-flash` → `gemini-2.5-pro` in order.

---

## Agent 2 — Visual Roadmap Agent

**File:** `server.ts` → `POST /api/generate-visual-roadmap`

**Purpose:** Generates a full node-graph roadmap (20–35 nodes, 25–40 edges) rendered by `@xyflow/react` on the frontend.

**Technique:** `generateObject` with an extended Zod schema covering node types, edge types, resources, and difficulty levels.

**Output schema (key fields):**
```typescript
z.object({
  nodes: z.array(z.object({
    id: z.string(),
    type: z.enum(["start","module","lesson","milestone","project","end"]),
    label: z.string(),
    description: z.string(),
    duration: z.string().optional(),
    difficulty: z.enum(["Beginner","Intermediate","Advanced"]).optional(),
    concepts: z.array(z.string()).optional(),
    resources: z.array(z.object({
      title: z.string(),
      type: z.enum(["video","article","doc","practice"]),
      url: z.string().optional(),
    })).optional(),
  })),
  edges: z.array(z.object({
    id: z.string(),
    source: z.string(),
    target: z.string(),
    type: z.enum(["required","optional","parallel"]),
  })),
})
```

**Graph rules enforced in prompt:**
- Exactly one `start` node (id: `"start"`)
- Exactly one `end` node (id: `"end"`)
- Milestones gate each module transition
- At least 2 project nodes (portfolio-worthy builds)
- Optional side-path nodes for going deeper
- First lesson: 20 minutes. Final project: 2–4 hours.
- **Strict Resource URL Safety:** The prompt strictly forbids fabricating specific deep-link URLs or deep paths. It enforces using stable official sites (e.g., developer.mozilla.org, docs.python.org, react.dev, freeCodeCamp, W3Schools, YouTube, GitHub) or falling back to homepage/search pages, explicitly outlawing dummy or placeholder domains such as `example.com`.

**Defensive Sanitization & Client Fallback:**
- Every generated node's resource URL is processed through `sanitizeResourceUrl` (defined in `src/lib/resource-link.ts`) before saving. If any URL matches known placeholder domains (e.g. `example.com`, `placeholder.com`, `yourdomain.com`) or is empty/malformed, it is automatically sanitized into a guaranteed-real Google Search query URL built dynamically from the resource title and type (`https://www.google.com/search?q=...`).
- The same utility is executed as a client-side layer in `VisualRoadmapGraph.tsx` to handle historical roadmaps in the database.

**Supports:** User's own Gemini API key via `x-user-key` header.

---

## Agent 3 — Lesson Agent

**File:** `server.ts` → `POST /api/generate-lesson`

**Purpose:** Generates a full markdown study guide for a specific lesson.

**Technique:** `generateText` — free-form markdown output, no schema required.

**Input:**
```typescript
{
  lessonTitle: string,
  courseTopic: string,
  concepts: string[],
  experienceLevel: string,
  backgroundContext?: string // optional learner background and preferences
}
```

**Output:** Raw markdown string — rendered in the frontend via `react-markdown`.

**Saved to:** `LessonContent` table in NeonDB (upserted per `courseId` + `lessonId`) via `trpc.saveLessonContent`, so the same lesson is never generated twice for the same user.

---

## Agent 4 — Quiz Agent

**File:** `server.ts` → `POST /api/generate-quiz`

**Purpose:** Generates exactly 3 adaptive multiple-choice questions for any lesson.

**Technique:** `generateObject` with a strict Zod schema — forces exactly 3 questions, each with 4 options and a correct index.

**Input:**
```typescript
{
  lessonTitle: string,
  lessonContent?: string   // capped at 600 chars to minimize tokens
}
```

**Output schema (Zod):**
```typescript
z.object({
  questions: z.array(z.object({
    id: z.string(),
    question: z.string(),
    options: z.array(z.string()).length(4),
    correctIndex: z.number().min(0).max(3),
    explanation: z.string(),
  })).length(3)
})
```

**Difficulty distribution enforced in prompt:** 1 easy, 1 medium, 1 hard.

**Score persistence:** Scores written to `Course.completedQuizzes` (JSON field) via `trpc.updateCourseProgress`.

**Fallback:** `getLocalFallbackQuiz()` — returns hardcoded MCQs if all models fail.

---

## Agent 5 — Mentor Agent (Agentic)

**File:** `server.ts` → `POST /api/mentor-chat`

**Purpose:** Acts as a persistent personal tutor. Answers questions, reads URLs, searches the web, and maintains full conversation history per course.

**Technique:** `generateText` with registered tools and an **agentic loop** (`stopWhen: isStepCount(5)`). The agent decides how many tool calls to make before responding — up to 5 steps per message.

**Tools:**

### `fetchUrl`
```typescript
tool({
  description: "Fetch and read the content of any URL...",
  inputSchema: z.object({
    url: z.string().url(),
    reason: z.string(),
  }),
  execute: async ({ url }) => {
    // SSRF protection: blocks 169.254.169.254, metadata.google.internal,
    // localhost, 127.0.0.1, 0.0.0.0, ::1, and all RFC-1918 private ranges
    // Protocol check: only http:// and https:// allowed
    // Fetches with 8s timeout, strips <script>, <style>, <nav>, <footer>
    // Returns clean text capped at 8000 chars
  }
})
```

### `searchWeb`
```typescript
tool({
  description: "Search the web using DuckDuckGo...",
  inputSchema: z.object({ query: z.string() }),
  execute: async ({ query }) => {
    // DuckDuckGo instant answer API — no API key required
    // Returns AbstractText + top 4 RelatedTopics
    // 5s timeout
  }
})
```

**Agentic loop flow:**
```
User message
    │
    ▼
Step 1: Model decides whether to call a tool
    │
    ├── Calls fetchUrl(url) → reads page → continues
    ├── Calls searchWeb(query) → reads results → continues
    └── Has enough context → generates final response
    │
    ▼
Steps 2–5: Can call more tools if needed
    │
    ▼
Final text response → saved to CourseMessage table
```

**Memory:** Full conversation history stored in `CourseMessage` table per course. Last 8 messages sent as context per request (token-optimized). Messages survive browser refresh, session expiry, and device switches.

**System prompt context:** Receives `currentCourseTitle` and `currentLessonTitle` on every call so responses are always scoped to what the user is studying.

**Model cascade:** `gemini-2.5-flash` → `gemini-2.0-flash` → `gemini-2.5-pro`. Retries on 503/429/overloaded errors with 1.5s sleep between attempts.

**Supports:** User's own Gemini API key via `x-user-key` header (bypasses server-side quota entirely).

---

## Agent 6 — Judge Agent

**File:** `server.ts` → `POST /api/generate-lesson` (integrated inline)

**Purpose:** Acts as an automated evaluation pipeline to score generated lesson content for quality (clarity, accuracy, depth, engagement) before it is sent to the user.

**Technique:** `generateObject` with a strict Zod schema for evaluation scores and feedback. Followed by a re-prompt of the Lesson Agent if the verdict is `needs_revision` or `fail` (max 1 retry to control latency).

**Input:**
```typescript
{
  lessonTitle: string,
  concepts: string[],
  contentToEvaluate: string
}
```

**Output schema (Zod):**
```typescript
z.object({
  clarityScore: z.number().min(1).max(10),
  accuracyScore: z.number().min(1).max(10),
  depthScore: z.number().min(1).max(10),
  engagementScore: z.number().min(1).max(10),
  overallScore: z.number().min(1).max(10),
  issues: z.array(z.string()),
  verdict: z.enum(["pass", "needs_revision", "fail"]),
  feedback: z.string(),
})
```

**Persistence:** The `overallScore` and `evaluationData` are returned alongside the lesson and saved via `trpc.saveLessonContent` to the `LessonContent` table for quality tracking.

---

## Agent 7 — Critic Agent

**File:** `server.ts` → `POST /api/generate-roadmap` and `POST /api/generate-visual-roadmap` (integrated inline)

**Purpose:** Acts as a quality control reviewer that examines a generated roadmap (standard or visual) before it reaches the user. If the roadmap is flawed (e.g. illogical ordering, bad time estimates, or invalid graph structure), the Critic Agent rejects it and provides revision notes, triggering a re-prompt of the original Roadmap Agent.

**Technique:** `generateObject` with a Zod schema to output structured critique (approval status, issues list, and revision notes). The Critic Agent runs synchronously before sending the response to the client.

**Input:**
```typescript
{
  // The full JSON output of the Roadmap Agent or Visual Roadmap Agent
}
```

**Output schema (Zod):**
```typescript
z.object({
  approved: z.boolean(),
  issues: z.array(z.object({
    severity: z.enum(["minor", "major"]),
    location: z.string(),
    problem: z.string(),
    suggestion: z.string(),
  })),
  revisionNotes: z.string().optional(),
})
```

**Persistence:** This agent runs inline and doesn't persist its critique directly to the database, though its presence is noted in a `_reviewMeta` object appended to the returned roadmap JSON.

---

## MCP Server — ZachCourse Tools

**File:** `mcp_server.ts`

**Purpose:** Exposes `fetchUrl` and `searchWeb` as a standalone MCP (Model Context Protocol) server so any MCP-compatible client (Claude Desktop, Cursor, etc.) can use ZachCourse's web tools externally.

**Transport:** `StdioServerTransport` — runs as a local process.

**Start:** `npm run mcp` (via `npx tsx mcp_server.ts`)

**Claude Desktop config:**
```json
{
  "mcpServers": {
    "zachcourse-tools": {
      "command": "npx",
      "args": ["tsx", "/path/to/zachcourse/mcp_server.ts"]
    }
  }
}
```

**Tools exposed:**
- `fetchUrl(url, reason)` — same SSRF protection as main server
- `searchWeb(query)` — DuckDuckGo instant answers

---

## Progress Agent — tRPC Layer

**File:** `src/server/trpc.ts`

**Purpose:** Not an AI agent — but acts as ZachCourse's **memory and state layer**. Reads and writes all learning progress to NeonDB, providing the persistent state that makes the AI agents context-aware across sessions.

**Procedures:**

| Procedure | Type | What it does |
|---|---|---|
| `getUserProgress` | query | Gets or creates `UserProgress` record |
| `updateUserProgress` | mutation | Updates streak, hours, quiz scores |
| `getCourses` | query | Lists all user's courses for sidebar |
| `getCourse` | query | Gets single course + last 50 messages |
| `createCourse` | mutation | Creates course + sends mentor welcome message |
| `updateCourseProgress` | mutation | Writes completed lessons + quiz scores |
| `addCourseMessage` | mutation | Persists a mentor chat message |
| `deleteCourse` | mutation | Deletes course (ownership-verified) |
| `renameCourse` | mutation | Renames course (ownership-verified) |
| `getLessonContent` | query | Fetches cached lesson markdown |
| `saveLessonContent` | mutation | Upserts generated lesson content |
| `saveVisualRoadmap` | mutation | Persists a visual roadmap |
| `updateVisualRoadmapProgress` | mutation | Updates completed node IDs in graph |
| `getModuleProject` | query | Fetches a generated project for a module |
| `generateProject` | mutation | Generates a beginner-friendly hands-on project using Gemini |
| `updateProjectStatus` | mutation | Updates a project's completion status and submission notes |
| `createCohort` | mutation | Creates a new learning cohort tied to a Course and/or VisualRoadmap |
| `previewCohortByInviteCode` | query | Fetches details of a cohort by code before joining |
| `joinCohortAndClone` | mutation | Enrolls user in a cohort and clones its assigned course/roadmap structure |
| `deleteCohort` | mutation | Deletes a cohort permanently if requested by the owner |
| `leaveCohort` | mutation | Leaves a joined cohort (for normal users/non-owners) |
| `getUserCohorts` | query | Gets cohorts the user has joined with progress details |
| `getCohortLeaderboard` | query | Computes leaderboard rankings scoped strictly to the cohort's content |
| `getCohortActivity` | query | Lists recent learning actions of members within the cohort |
| `createClassroom` | mutation | Creates a teacher classroom linked to specific curriculum |
| `getTeacherClassrooms` | query | Lists classrooms managed by the current teacher |
| `getClassroomRoster` | query | Returns all student members in a classroom with scoped progress metrics |
| `getStudentDetail` | query | Returns a detailed breakdown of courses and weak topics for a student |

**Auth:** All procedures use `protectedProcedure` — throws `UNAUTHORIZED` if no active session.

---

## Cohort & Classroom Architecture

To prevent clutter and ensure that progress metrics remain highly contextual and fair, every **Cohort** and **Classroom (Cohort as a Class)** is strictly tied to a specific `Course` and/or `VisualRoadmap`.

```
Cohort / Classroom (Owner/Teacher-controlled)
         │
         ├── linked Course ──► [Sourced from Owner/Teacher]
         └── linked VisualRoadmap ──► [Sourced from Owner/Teacher]
         
         Joining/Enrolling Students (Learner View)
         │
         ├── Clones Course (with clean progress: completedLessons: [], completedQuizzes: {})
         └── Clones VisualRoadmap (with clean progress: completedNodeIds: [])
```

### Key Architectural Rules:
1. **At Least One Content Resource:** A cohort must specify either a valid Course ID or a VisualRoadmap ID owned by the creator.
2. **Clone-on-Join Flow:** When a user joins via an invite code, they preview the content first. Upon confirmation, the system clones the course/roadmap skeleton into their private profile (`clonedFromCourseId`/`clonedFromRoadmapId` references). This ensures each learner tracks their own progress starting at 0% while sharing the exact same curriculum structure.
3. **No Lesson-Text Sharing:** To respect individual learning styles and levels, generated `LessonContent` is never shared or pre-copied. Each user triggers their own Lesson Agent on demand.
4. **Scoped Leaderboard & Metrics:** Leaderboards and classroom rosters dynamically calculate member proficiency, average quiz scores, and streak counts *strictly scoped* to the cloned copies belonging to that cohort’s assigned course or roadmap. Overall platform-wide statistics are filtered out.

---

## Security

| Layer | Mechanism |
|---|---|
| Route auth | `requireAuth` middleware on all AI routes — Better Auth session check |
| Rate limiting | `express-rate-limit`: 20 req/min per IP on all AI endpoints |
| Body size | `express.json({ limit: "2mb" })` |
| Input validation | Max 500 chars topic, 2000 chars message, 10000 chars textContent |
| SSRF protection | `fetchUrl` blocks private IPs, metadata endpoints, non-http protocols |
| CORS protection | Strict origin verification. Origins must strictly match `ALLOWED_ORIGINS` (including deployed URLs) or localhost; wildcard suffix matches on shared `*.run.app` domains are explicitly disabled to prevent session-cookie leakage across distinct services. |
| HTTP headers | `helmet` — sets XSS, clickjacking, MIME sniffing protection headers. Connect-src is customized to permit `https://generativelanguage.googleapis.com` for client-side direct API key interactions, maintaining robust CSP controls. |
| Ownership checks | tRPC mutations verify `course.userId === ctx.user.id` before any write/delete |

---
> Source: [19akshansh/zachcourse](https://github.com/19akshansh/zachcourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
