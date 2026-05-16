## ngx-transform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Versión:** 11.0 (Genesis Doctrine)
**Stack:** Next.js 16.0.7 + React 19 + TypeScript + Firebase + Tailwind CSS v4 + Upstash Redis

NGX Transform is a **premium viral lead magnet** that creates realistic 12-month physical transformation projections. Users upload a photo, provide profile data, and receive AI-generated insights with visualized progress images at m0/m4/m8/m12 milestones, plus a personalized 7-day fitness plan.

**v11.0 Genesis Doctrine**: GENESIS is the **sole AI entity visible to the user**. Internally, 13 specialized modules power 4 capabilities (Entrenamiento, Nutricion, Recuperacion, Habitos). Users never see module names — only "GENESIS" and its capabilities.

## Business Context

**Strategic Purpose**: NGX Transform is a **viral lead-generation tool** designed to capture users and convert them to NGX's main subscription fitness app.

**Growth Strategy**:
- Free tool with high shareability (transformation results are inherently viral)
- Captures email leads at wizard entry
- Results page includes CTA to NGX subscription app
- Shareable URLs with Open Graph meta tags for social spread

**Differentiators**:
- Not just "before/after" - temporal projection with narrative (m0→m4→m8→m12)
- Mental + Physical analysis (stress, sleep, discipline factors)
- Cinematic "Nike commercial" aesthetic vs generic fitness apps
- Uses YOUR actual photo, not generic avatars

## User Journey

```
Landing (/)
    ↓
Wizard (/wizard)
    ├── Email capture (lead)
    ├── Photo upload (dropzone)
    ├── Identity & Biometrics (age, sex, height, weight, bodyType)
    ├── Focus Zone (upper/lower/abs/full)
    ├── Goals & Strategy (level, goal, weeklyTime)
    └── Mental Logs (stress, sleep, discipline sliders)
    ↓
Processing (BiometricLoader)
    ├── "Iniciando escaneo biométrico..."
    ├── "Analizando densidad muscular..."
    ├── "Proyectando estructura ósea..."
    └── Motivational tips rotation
    ↓
Results (/s/[shareId])
    ├── CinematicViewer (fullscreen immersive)
    ├── Timeline navigation (HOY → MES 4 → MES 8 → MES 12)
    ├── TransformationSummary (stats delta)
    ├── NeonRadar stats visualization
    └── CTA: "Ver cómo GENESIS crea tu plan"
    ↓
Genesis Demo (/s/[shareId]/demo) — v11.0
    ├── AgentOrchestration (GENESIS central + 4 capability cards)
    │   ├── Phase 1: Analyzing profile (internal: GENESIS, STELLA, LOGOS)
    │   ├── Phase 2: Entrenamiento + Nutricion progress (internal: BLAZE, TEMPO, ATLAS, SAGE, MACRO, METABOL)
    │   └── Phase 3: Recuperacion + Habitos progress (internal: WAVE, SPARK, NOVA, LUNA)
    └── DemoChat (5 interactions, all from GENESIS with capability labels)
    ↓
Plan Preview (/s/[shareId]/plan)
    ├── Day 1: Complete and functional (WorkoutCard, MealPlan, Checklist)
    ├── Days 2-7: Blurred + locked
    └── ComparisonCTA ("Sin GENESIS vs Con GENESIS")
    ↓
Conversion
    └── CTA: "DESBLOQUEAR MI PLAN COMPLETO"
```

## Development Commands

All commands run from `app/`:

```bash
cd app
pnpm dev          # Dev server at localhost:3000
pnpm build        # Production build
pnpm start        # Production server
pnpm lint         # ESLint
```

## Architecture

**Stack**: Next.js 16.0.7 (App Router) + React 19 + Firebase + Google Gemini + Tailwind CSS v4

### Data Flow

1. **Wizard** (`/wizard`) → User uploads photo + profile data
2. **Session Creation** → Firebase Storage (photo) + Firestore (session doc)
3. **Analysis** (`/api/analyze`) → Gemini generates insights with 4-stage timeline
4. **Image Generation** (`/api/generate-images`) → Gemini Image API creates m4/m8/m12 transformations
5. **Results** (`/s/[shareId]`) → Shareable results page with timeline viewer

### Key Services

| Service | Location | Purpose |
|---------|----------|---------|
| `gemini.ts` | `src/lib/` | Gemini 2.5 Flash for profile analysis + user_visual_anchor + style_profile |
| `nanobanana.ts` | `src/lib/` | Gemini Image API with Identity Chain for consistent transformations |
| `firebaseAdmin.ts` | `src/lib/` | Server-side Firestore/Storage operations |
| `storage.ts` | `src/lib/` | Signed URL generation, buffer uploads |
| `validators.ts` | `src/lib/` | Zod schemas for all API inputs |
| `telemetry.ts` | `src/lib/` | Funnel event tracking (wizard_start → cta_completed) |
| `jobManager.ts` | `src/lib/` | Idempotent, resumable jobs for image generation |
| `imageConfig.ts` | `src/lib/` | Centralized image config (model, aspect ratio, size) |
| `promptBuilder.ts` | `src/lib/` | Robust prompt constructor with Identity Lock |
| `qualityGates.ts` | `src/lib/` | Output validation (face visible, single subject, no artifacts) |
| `viral/*` | `src/lib/viral/` | Share-to-unlock, referral tracking, social pack generator |
| `plan/*` | `src/lib/plan/` | 7-day plan generation with AI + templates |
| `schemas/*` | `src/lib/schemas/` | Strict Zod schemas for analysis output |
| `firebaseClient.ts` | `src/lib/` | Client-side Firebase SDK (auth + storage) |
| `emailScheduler.ts` | `src/lib/` | Email nurture sequence scheduling (D0-D7) |
| `watermark.ts` | `src/lib/` | Image watermarking with Sharp |
| `utils.ts` | `src/lib/` | General utility functions (cn, formatters) |
| `rateLimit.ts` | `src/lib/` | Distributed rate limiting with Upstash Redis (v3.0) |
| `genesis-demo/agents.ts` | `src/lib/` | Internal module config, capability mapping, orchestration phases |
| `genesis-orchestrator.ts` | `src/lib/` | GENESIS orchestration logic for chat responses |
| `authServer.ts` | `src/lib/` | Server-side Firebase auth (requireAuth, token validation) |
| `spendLimiter.ts` | `src/lib/` | Firestore-based AI spend limiter ($50/day, $10/hour) |
| `aiKillSwitch.ts` | `src/lib/` | AI safety kill switch (emergency spending halt) |
| `elevenlabs-voice.ts` | `src/lib/` | ElevenLabs TTS integration for voice agent |
| `emailSuppression.ts` | `src/lib/` | Email opt-out/suppression management |

### API Routes

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/sessions` | POST | Create session with profile + photo |
| `/api/sessions/[shareId]` | GET | Fetch session data |
| `/api/sessions/[shareId]/urls` | GET | Get signed URLs for images |
| `/api/analyze` | POST | Run Gemini analysis on session |
| `/api/generate-images` | POST | Generate m4/m8/m12 transformation images |
| `/api/email` | POST | Send results via Resend |
| `/api/leads` | POST | Capture email leads |
| `/api/unlock` | POST | Share-to-unlock flow (share_intent, request_unlock) |
| `/api/referral` | POST | Referral tracking (visit, complete, claim, stats) |
| `/api/social-pack/[shareId]` | GET | Download social pack (story, post, square) |
| `/api/plan` | POST/GET | Generate or fetch 7-day personalized plan |
| `/api/og/[shareId]` | GET | OG image (split-screen HOY vs 12 MESES) |
| `/api/counter` | GET/POST | Social proof counter (weekly transformations) |
| `/api/email/sequence` | POST | Email nurture sequence management |
| `/api/email/send` | POST | Send specific email template (D0-D7) |
| `/api/genesis-demo` | GET | SSE streaming for agent orchestration animation (v3.0) |
| `/api/genesis-chat` | POST | DemoChat responses with A2UI widgets (v3.0) |
| `/api/genesis-voice` | POST | Voice agent responses via ElevenLabs (v3.0) |
| `/api/remarketing` | POST/GET | Remarketing leads (POST: register, GET: admin lookup) |
| `/api/generate-plan` | GET/POST | PDF plan generation with rate limiting |
| `/api/csp-report` | POST | CSP violation reports |
| `/api/sessions/[shareId]/private` | GET | Authenticated session data (full PII) |
| `/api/sessions/[shareId]/share-settings` | POST | Update share scope (shareOriginal, shareInsights, shareProfile) |
| `/api/sessions/me` | GET | List sessions for authenticated user |
| `/api/cron/cleanup` | POST | Cleanup expired sessions/orphaned assets |
| `/api/health` | GET | Uptime health check |
| `/api/unsubscribe` | POST | Email suppression (unsubscribe/resubscribe) |

### Type Definitions

Core AI types in `src/types/ai.ts`:
- `InsightsResult` - Full analysis output with timeline
- `TimelineEntry` - Single milestone (m0/m4/m8/m12) with stats, prompts, mental notes
- `OverlayPoint` - Hotspot coordinates for image annotations
- `UserVisualAnchor` - Immutable description of facial/physical traits for identity preservation
- `StyleProfile` - Visual style parameters (lighting, wardrobe, background, color_grade)
- `LetterFromFuture` - m12 motivational letter content

Plan types in `src/lib/plan/planTypes.ts`:
- `SevenDayPlan` - Complete 7-day plan structure
- `DayPlan` - Single day with workout, habits, nutrition, mindset
- `Exercise` - Exercise definition with sets, reps, rest, notes

### Profile Schema (Mental Logs)

The wizard captures extended profile data beyond basic biometrics:

```typescript
// Core biometrics
age, sex, heightCm, weightKg

// Training context
level: "novato" | "intermedio" | "avanzado"
goal: "definicion" | "masa" | "mixto"
weeklyTime: 1-14 hours

// Body classification
bodyType: "ectomorph" | "mesomorph" | "endomorph"
focusZone: "upper" | "lower" | "abs" | "full"

// Mental Logs (differentiator)
stressLevel: 1-10      // Affects recovery recommendations
sleepQuality: 1-10     // Impacts training intensity suggestions
disciplineRating: 1-10 // Influences consistency expectations
```

The **Mental Logs** are fed to Gemini's "Elite Coach" prompt to personalize recommendations based on lifestyle factors, not just physical metrics.

### Key UI Components

| Component | Location | Purpose |
|-----------|----------|---------|
| `CinematicViewer` | `src/components/` | Fullscreen immersive results display with Nike-style aesthetic |
| `BiometricLoader` | `src/components/` | Animated loading screen with scanning effect and tips |
| `NeonRadar` | `src/components/` | Radar chart for strength/aesthetics/endurance/mental stats |
| `OverlayImage` | `src/components/` | Interactive image with clickable hotspots |

### Results 2.0 Components (v2.0)

| Component | Location | Purpose |
|-----------|----------|---------|
| `CinematicAutoplay` | `src/components/results/` | Animated reveal sequence m0→m4→m8→m12 |
| `CompareSlider` | `src/components/results/` | Touch-friendly before/after slider |
| `LetterFromFuture` | `src/components/results/` | Modal with typewriter animation for m12 letter |
| `StatsDelta` | `src/components/results/` | Animated stats progression with delta indicators |
| `ChapterView` | `src/components/results/` | Detailed milestone view with hero, narrative, stats |
| `TransformationViewer2` | `src/components/` | Orchestrator for Results 2.0 experience |
| `ShareToUnlock` | `src/components/viral/` | Share-to-unlock UI with countdown |

### Viral Optimization Components (v2.1)

| Component | Location | Purpose |
|-----------|----------|---------|
| `DramaticReveal` | `src/components/results/` | Countdown + slow morph reveal (m0→m4→m8→m12) |
| `ShareToUnlockModal` | `src/components/viral/` | Modal for share-to-unlock premium content |
| `SocialCounter` | `src/components/` | Weekly transformation counter with social proof |
| `AgentBridgeCTA` | `src/components/` | GENESIS CTA with capability-based context selection |
| `ReferralCard` | `src/components/` | Referral code UI with copy functionality |

### Genesis Experience Components (v11.0)

| Component | Location | Purpose |
|-----------|----------|---------|
| `AgentOrchestration` | `src/components/genesis/` | GENESIS central + 4 capability cards with SSE-driven animation |
| `DemoChat` | `src/components/genesis/` | Chat (5 interactions) — all messages from GENESIS with capability labels |
| `PlanPreview` | `src/components/genesis/` | Day 1 visible, Days 2-7 blurred/locked |
| `AgentStatusBar` | `src/components/demo/` | GENESIS capability progress indicators |
| `GenesisChat` | `src/components/demo/` | SSE chat — messages as "GENESIS: Optimizando entrenamiento" |
| `TransformationSummary` | `src/components/results/` | Timeline + stats delta after CinematicViewer |
| `ComparisonCTA` | `src/components/results/` | "Sin GENESIS vs Con GENESIS" comparison table |

### A2UI Widgets (v3.0) ⭐ NEW

Ported from genesis_A2UI for consistent agent-style UI:

| Widget | Location | Purpose |
|--------|----------|---------|
| `GlassCard` | `src/components/widgets/` | Base card with glassmorphism + capability accent |
| `AgentBadge` | `src/components/widgets/` | GENESIS capability badge (colored uppercase pill) |
| `ActionButton` | `src/components/widgets/` | Full-width gradient CTA button |
| `ProgressBar` | `src/components/widgets/` | Animated progress with capability color |
| `WorkoutCard` | `src/components/widgets/` | Exercise display with sets/reps/rest |
| `MealPlan` | `src/components/widgets/` | Nutrition plan with macros |
| `InsightCard` | `src/components/widgets/` | GENESIS insight with icon + message |
| `ChecklistWidget` | `src/components/widgets/` | Interactive checklist with completion |

### GENESIS Architecture (v11.0)

**GENESIS** (`#6D00FF`, Brain icon) is the sole user-facing AI entity. Internally, 13 modules are grouped into 4 capabilities:

| Capability | Color | Icon | Internal Modules |
|------------|-------|------|------------------|
| Entrenamiento | `#fb923c` | Flame | BLAZE, ATLAS, TEMPO |
| Nutricion | `#34d399` | Leaf | SAGE, MACRO, METABOL |
| Recuperacion | `#60a5fa` | Timer | WAVE, NOVA, LUNA |
| Habitos | `#a78bfa` | Sparkles | SPARK, STELLA, LOGOS |

**Key types** (`src/types/genesis.ts`):
- `GenesisCapability`: `'entrenamiento' | 'nutricion' | 'recuperacion' | 'habitos'`
- `AgentType`: Internal module identifiers (legacy, not user-facing)

**Rules**:
- All user-facing communication comes from "GENESIS"
- Messages use first person: "Estoy analizando..." (never "BLAZE reporta:")
- Capability labels appear as context: "GENESIS · Entrenamiento"
- Module names (BLAZE, SAGE, etc.) exist only in internal config and SSE compatibility layers

### Email Nurture Sequence (v2.1)

| Template | Location | Timing | Purpose |
|----------|----------|--------|---------|
| `D0Results` | `src/emails/sequence/` | Immediate | Results ready notification |
| `D1Reminder` | `src/emails/sequence/` | Day 1 | Reminder to view full analysis |
| `D3Plan` | `src/emails/sequence/` | Day 3 | 7-day plan introduction |
| `D7Conversion` | `src/emails/sequence/` | Day 7 | NGX ASCEND conversion CTA |

### AI Prompt Strategy

The Gemini integration uses an **"Elite High-Performance Coach & Futurist"** persona:
- Stoic, clinical, yet motivational tone
- Generates 4-stage timeline (m0/m4/m8/m12) with:
  - `title`: Phase name (e.g., "GÉNESIS", "METAMORFOSIS")
  - `description`: Clinical summary of changes
  - `stats`: Numerical scores (0-100) for Strength, Aesthetics, Endurance, Mental
  - `image_prompt`: Detailed prompt for Nike-style transformation image
  - `mental`: Stoic mindset shift required for each stage
- Image generation uses "Nike Advertisement" / "Billboard Campaign" style prompts

### Identity Chain (v2.0)

For consistent facial identity across m4/m8/m12 transformations:

```
[IDENTITY LOCK]
Subject: {user_visual_anchor}
PRESERVE: exact facial features, skin tone, hair color/style, distinguishing marks

[TRANSFORMATION: {step} - {percent}%]
Environment:
- m4: gritty underground gym (40% progress)
- m8: lifestyle setting (70% progress)
- m12: editorial studio (100% peak form)

[NEGATIVE]
NO: CGI, cartoon, plastic skin, extra limbs, face drift, multiple subjects
```

**Reference Chaining:**
- m4 refs: [original, styleRef]
- m8 refs: [original, styleRef, m4]
- m12 refs: [original, styleRef, m8]

### Feature Flags (v3.0)

| Flag | Default | Purpose |
|------|---------|---------|
| `NEXT_PUBLIC_FF_RESULTS_2` | false | Enable Results 2.0 experience |
| `FF_OG_SPLIT_SCREEN` | true | OG split-screen (HOY vs 12 MESES) |
| `FF_SHARE_TO_UNLOCK` | true | Unlock content after sharing |
| `FF_REFERRAL_TRACKING` | true | Track referral visits/completions |
| `FF_PLAN_7_DIAS` | true | Generate 7-day plan with AI |
| `FF_DRAMATIC_REVEAL` | true | Enable dramatic countdown reveal |
| `FF_SOCIAL_COUNTER` | true | Show weekly transformation counter |
| `FF_AGENT_BRIDGE_CTA` | true | Show GENESIS capability-based CTA |
| `FF_EMAIL_SEQUENCE` | true | Enable D0-D7 email nurture sequence |
| `FF_TELEMETRY_ENABLED` | true | Enable funnel telemetry (v3.0) |
| `FF_DELETE_TOKEN_REQUIRED` | true | Require delete token for session deletion (v3.0) |
| `FF_NB_PRO` | false | Enable Nano Banana Pro (Gemini 3 Pro Image) |
| `FF_IDENTITY_CHAIN` | true | Enable identity chain for consistent faces |
| `FF_QUALITY_GATES` | true | Enable output validation gates |
| `FF_CINEMATIC_AUTOPLAY` | true | Auto-play cinematic reveal on results |
| `FF_COMPARE_SLIDER` | true | Enable before/after comparison slider |
| `FF_LETTER_FROM_FUTURE` | true | Enable m12 motivational letter modal |
| `FF_EXPOSE_ORIGINAL` | true | Expose original uploaded photo in public results |

### Page Routes

| Route | Purpose |
|-------|---------|
| `/` | Landing page |
| `/wizard` | Multi-step wizard (email, photo, profile) |
| `/s/[shareId]` | Shareable results page with TransformationSummary |
| `/s/[shareId]/demo` | Genesis Experience (GENESIS + 4 capabilities orchestration + chat) |
| `/s/[shareId]/plan` | Plan preview (Day 1 free, Days 2-7 locked) |
| `/plan/[shareId]` | Legacy redirect → `/s/[shareId]/plan` |
| `/auth` | Firebase auth callback |
| `/dashboard` | User dashboard (session history) |
| `/dashboard/[shareId]` | Individual session management |
| `/account` | Account settings |
| `/loading/[shareId]` | Loading experience with progress animation |
| `/demo/[shareId]` | Legacy redirect → `/s/[shareId]/demo` |
| `/j` | Short link redirect |
| `/m` | Marketing redirect |
| `/email/preview` | Email template preview (dev only) |
| `/privacy` | Privacy policy |
| `/terms` | Terms of service |
| `/unsubscribe` | Email unsubscribe page |

## Environment Variables

Copy `app/.env.example` to `app/.env.local`:

```bash
# Required - Firebase
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY="-----BEGIN..."  # Escape \n as literal

# Required - AI
GEMINI_API_KEY=

# Required - Security (v3.0)
CRON_API_KEY=                    # API key for internal endpoints (counter, email, remarketing)

# Required - Rate Limiting (v3.0)
UPSTASH_REDIS_REST_URL=          # From https://console.upstash.com/redis
UPSTASH_REDIS_REST_TOKEN=        # From https://console.upstash.com/redis

# Optional - AI Models
GEMINI_IMAGE_MODEL=gemini-3.1-flash-image-preview  # Default image model (NanoBanana 2)
GEMINI_IMAGE_MODEL=gemini-3-pro-image-preview      # Pro image model (Identity Chain, FF_NB_PRO=true)
PLAN_GENERATION_MODEL=gemini-2.5-flash             # Plan generation model

# Optional - Email
RESEND_API_KEY=                  # For email sharing

# Optional - Booking
NEXT_PUBLIC_BOOKING_URL=         # CTA link
```

## Design System

**Colors**: Electric Violet `#6D00FF` (primary), Deep Purple `#5B21B6` (accent), Background `#030005`

**CSS Variables**: Defined in `globals.css`, exposed via `--primary`, `--accent`, `--ngx-electric-violet`

**Fonts**: Space Grotesk (primary, via next/font), Neue Haas Grotesk Text Pro (self-hosted fallback), United Sans (display headings)

**Components**: shadcn/ui in `src/components/ui/` and `src/components/shadcn/ui/`

## File Conventions

- Components: `PascalCase.tsx` in `src/components/`
- Lib/utils: `camelCase.ts` in `src/lib/`
- API routes: `route.ts` following Next.js App Router patterns
- Path alias: `@/*` maps to `./src/*`

## Security (v3.0) ⭐ NEW

### API Protection

| Endpoint | Protection |
|----------|------------|
| `DELETE /api/sessions/[shareId]` | Delete token validation (via `X-Delete-Token` header or `?token=` query) |
| `POST /api/email` | Only sends to session owner's email (prevents spam vector) |
| `POST /api/counter` | Requires `CRON_API_KEY` (prevents counter manipulation) |
| `GET /api/remarketing` | Requires `CRON_API_KEY` (protects PII) |
| `POST /api/remarketing` | Upstash rate limiting by email |
| `GET /api/generate-plan` | Upstash rate limiting by IP |

### Rate Limiting (Upstash Redis)

Distributed rate limiting via `src/lib/rateLimit.ts`:

```typescript
// Rate limits by endpoint
"api:sessions"      // 3 per day (by IP)
"api:remarketing"   // 3 per minute (by email)
"api:generate-plan" // 10 per hour (by IP)
"api:email"         // 2 per hour (by IP)
```

### Content Security Policy (CSP)

Implemented in `src/middleware.ts`:

- **script-src**: `'self'` + nonce-based + `'strict-dynamic'`
- **connect-src**: Whitelisted Firebase, Upstash, Vercel, Google APIs
- **frame-ancestors**: `'self'` only
- **object-src**: `'none'`
- **upgrade-insecure-requests**: Enabled

### Security Headers

| Header | Value |
|--------|-------|
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains; preload` |
| `X-Frame-Options` | `SAMEORIGIN` |
| `X-Content-Type-Options` | `nosniff` |
| `X-XSS-Protection` | `1; mode=block` |
| `Referrer-Policy` | `origin-when-cross-origin` |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` |

### Share Scope System

Sessions support granular share scopes:
- `shareOriginal` — allow sharing of original uploaded photo
- `shareInsights` — allow sharing of AI analysis insights
- `shareProfile` — allow sharing of profile data

Managed via `POST /api/sessions/[shareId]/share-settings`. Public `GET /api/sessions/[shareId]` strips PII to allowlisted fields only.

### Authentication & Authorization

- `requireAuth()` mandatory on `/api/analyze` and `/api/generate-images`
- `acquireJobLock()` for atomic job concurrency (prevents duplicate image generation)
- Delete token validation via `X-Delete-Token` header or `?token=` query param

### Telemetry Events

| Event | Trigger |
|-------|---------|
| `rateLimitBlocked` | Request exceeds Upstash rate limit |
| `authFailed` | Missing/invalid auth on protected endpoint |
| `spendLimitBlocked` | AI spend limit exceeded |
| `jobLockDenied` | Concurrent job lock acquisition failed |

### Delete Token Flow

```
1. Session creation → Generate deleteToken
2. Store: Firestore + return in response
3. DELETE request → Validate token via header/query
4. Invalid token → 403 Forbidden
```

## Architecture Decisions (v3.0)

### Why Upstash vs In-Memory Rate Limiting?

**Problem**: In-memory `Map` resets on cold starts, bypassed across Vercel instances.

**Solution**: Upstash Redis with sliding window algorithm:
- Distributed across all instances
- Persists across deployments
- Sub-50ms latency via REST API
- Automatic cleanup (no TTL management)

**Note**: Upstash Redis runs on **free tier** (10K ops/day, ~1,600 users/day). Cost: $0. The real cost protection is the Firestore `gemini_spend` limiter ($50/day, $10/hour), not Redis. If Redis goes down, non-critical endpoints fall back to in-memory; critical endpoints fail closed (fail-safe). Redis is prescindible but free, so we keep it.

### Why SSE for Agent Orchestration?

**Problem**: WebSocket overkill for unidirectional event stream.

**Solution**: Server-Sent Events (SSE):
- Native browser support
- No WebSocket infrastructure
- Simple reconnection handling
- Works through proxies/CDNs

### Genesis Demo Flow (v11.0)

```
/api/genesis-demo (SSE)
    ├── Event: phase (phase 1 start)
    ├── Event: agent (module status — internal)
    │   ... (20-25 seconds, 13 modules across 3 phases)
    └── Event: complete

Client → AgentOrchestration.tsx (v11.0)
    ├── Subscribe to SSE
    ├── Map module events → capability states
    │   (e.g., BLAZE complete → Entrenamiento progress++)
    ├── Show GENESIS central + 4 capability progress cards
    └── Show DemoChat when complete
         └── All messages: "GENESIS · {Capability}"
```

---
> Source: [270aldo/ngx-transform](https://github.com/270aldo/ngx-transform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
