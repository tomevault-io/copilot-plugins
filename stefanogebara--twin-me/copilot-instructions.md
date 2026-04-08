## twin-me

> - **Crons**: NEVER more than */15. Removed token-refresh cron (on-demand only). deliver-insights and prospective-check at */15.

# TwinMe - Soul Signature Platform

## Vercel Cost Rules (CRITICAL — $375 bill incident March 2026)

- **Crons**: NEVER more than */15. Removed token-refresh cron (on-demand only). deliver-insights and prospective-check at */15.
- **maxDuration**: 60s (was 120s — halves GB-hour cost)
- **Deploys**: ONE per push (disabled GitHub Action duplicate). Batch commits before pushing.
- **New crons**: Must justify frequency. Default to hourly or daily, not every-N-minutes.
- **LLM in crons**: Always check cooldowns/conditions BEFORE calling LLM. Early return = free.

## Workflow Orchestration

### 1. Plan Node Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One tack per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Bug Fixing: Test-First
- When a bug is reported, do NOT start by trying to fix it
- First, write a test that reproduces the bug (the test must FAIL)
- Then, launch subagents to fix the bug and prove it with a passing test
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---

## Vision
TwinMe creates digital twins that capture your true originality - your **Soul Signature**. We go deeper than public information by discovering what makes you authentically YOU through the digital footprints that reveal your genuine curiosities, passions, and patterns.

**One-liner:** A data-driven personality portrait that reveals patterns about yourself you never noticed, powers an AI twin that actually knows you, and lets you share your authentic self.

## Core Product Loop
```
1. ONBOARDING    -> Cofounder.co-style: email lookup -> instant wow -> interactive Q&A
2. SOUL SIGNATURE -> Cross-platform personality portrait from real data
3. TWIN CHAT     -> AI twin that embodies your personality and knows your life
```

## Twin Architecture (Generative Agents-Inspired)

Based on Park et al., UIST 2023 ("Generative Agents: Interactive Simulacra of Human Behavior").

### Memory Stream (`user_memories` table)
Single unified store for ALL memory types:
- **Observations** - Raw platform data (Spotify plays, calendar events, YouTube activity)
- **Conversations** - Per-utterance chat exchanges with the twin
- **Facts** - Extracted facts about the user
- **Reflections** - Higher-level synthesized insights (stored back as memories)

Each memory has: content, embedding (1536d vector), importance_score (1-10), created_at, last_accessed_at.

### Retrieval (`search_memory_stream` RPC)
Three-factor scoring with context-dependent weights and min-max normalization:
```
score = w_recency * norm(recency) + w_importance * norm(importance) + w_relevance * norm(relevance)
```
- **Recency**: `0.995^hours_since_last_access` (accessing refreshes the timestamp)
- **Importance**: LLM-rated 1-10 at creation time
- **Relevance**: Cosine similarity of embeddings

Weight presets (inspired by Paper 2):
- `default` [1.0, 1.0, 1.0] - General conversation (original Generative Agents)
- `identity` [0.2, 0.8, 1.0] - Twin summary, personality queries (relevance dominant)
- `recent` [1.0, 0.5, 0.7] - Proactive insights, "what's happening" queries
- `reflection` [0.0, 0.5, 1.0] - Expert reflections (Paper 2 style: no recency bias)

### Expert Reflection Engine (`reflectionEngine.js`)
Inspired by Paper 2 (Park et al., 2024 "Generative Agent Simulations of 1,000 People").
Triggered when accumulated importance > 40 (IMPORTANCE_THRESHOLD in reflectionEngine.js). Uses 5 domain-specific expert personas:

1. **Personality Psychologist** - Emotional patterns, coping, attachment style, Big Five from behavior
2. **Lifestyle Analyst** - Daily rhythms, energy, health-behavior connections, routine vs spontaneity
3. **Cultural Identity Expert** - Aesthetic preferences, media taste, cultural markers
4. **Social Dynamics Analyst** - Communication style, relationship patterns, social energy
5. **Motivation Analyst** - Work patterns, ambitions, decision-making style

Process: Gather 100 recent memories -> Run all 5 experts in parallel (each retrieves domain-specific evidence via vector search with `reflection` weights) -> Each expert generates 2-3 observations -> Store as `reflection` memories (importance 7-9) with expert metadata -> Recursive up to depth 3

### Background Observation Ingestion
Periodic cron job pulls platform data and stores as observations:
```
Platform APIs -> Natural language observations -> Memory Stream
                                                    |
                                              [Importance accumulates]
                                                    |
                                              Reflection Engine triggers
                                                    |
                                              Proactive Insights generated
```

### Dynamic Twin Summary (`twinSummaryService.js`)
Periodically regenerated summary replacing static soul signature:
- Five parallel retrieval queries aligned to expert domains: personality, lifestyle, cultural identity, social dynamics, motivation
- Uses `identity` retrieval weights (relevance dominant, low recency bias)
- Cached in `twin_summaries` table with 4-hour TTL
- Upserts on `user_id` conflict - one summary per user
- Regenerated after significant memory accumulation

### Proactive Insights (`proactiveInsights.js`)
TwinMe's equivalent of the paper's Planning system - the twin notices things and brings them up:
- Triggered after observation ingestion
- LLM analyzes recent memories + reflections to generate 1-3 insights
- Stored in `proactive_insights` table with urgency (high/medium/low) and category
- Injected into twin chat as "THINGS I NOTICED" context section
- Marked `delivered` after being included in a twin response
- High urgency sorted first for delivery priority

### Twin-Driven Goal Tracking (`goalTrackingService.js`)
The twin observes platform data patterns and SUGGESTS achievable goals. Once accepted, progress is auto-tracked from platform data and the twin weaves accountability into conversations naturally.

**Tables**: `twin_goals`, `goal_progress_log` (migration: `20260220_create_twin_goals.sql`)
**API**: `api/routes/goals.js` (7 endpoints under `/api/goals`)
**Frontend**: `src/pages/GoalsPage.tsx` + components in `src/pages/components/goals/`

Flow: Observation ingestion -> `generateGoalSuggestions()` -> user accepts -> `trackGoalProgress()` auto-tracks -> twin references in chat -> celebration on completion

**Metric extraction**: Primary from structured platform data, fallback to regex on memory stream text (reflections dominate recent memories ~90%, so scan 200+ entries to find platform_data).

### Soul Signature Voting Layer (`personalityProfileService.js`)
Neural-inspired personality shaping for twin responses. Based on CL1_LLM_Encoder's biological neuron blending concept (`blended[tok] = (1-alpha) * model_probs + alpha * neural_probs`), adapted for API-level constraints (Claude via OpenRouter does NOT expose logprobs).

**Three-layer intervention:**
1. **Prompt Injection** (0x cost) - OCEAN Big Five personality traits + stylometric fingerprint translated into behavioral instructions injected into system prompt via `personalityPromptBuilder.js`
2. **Sampling Parameters** (0x cost) - OCEAN dimensions mapped to temperature (0.5-0.9), top_p (0.85-0.95), frequency_penalty (0-0.3), presence_penalty (0-0.3)
3. **Best-of-N Reranking** (3x cost, feature-flagged via `ENABLE_PERSONALITY_RERANKER`) - Generate N candidates with temperature spread, embed all, select by cosine similarity to personality embedding centroid

**Personality Profile** (`user_personality_profiles` table):
- OCEAN Big Five scores (0-1) extracted via LLM analysis (TIER_ANALYSIS) of reflections + conversations + facts
- Stylometric fingerprint: sentence length, vocabulary richness (type-token ratio), formality, emotional expressiveness, humor markers, punctuation distribution — all pure computation, no LLM
- Derived sampling parameters from OCEAN mapping
- Personality embedding centroid: weighted average of memory embeddings with 7-day recency half-life and importance weighting
- Profile TTL: 12 hours, auto-rebuild when stale. Min 20 memories required.

**OCEAN-to-Sampling Mapping:**
- High Openness → higher temperature (more creative), wider top_p
- High Conscientiousness → lower temperature (more precise), narrower top_p
- High Extraversion → higher presence_penalty (explores topics), higher frequency_penalty (varied vocabulary)
- High Agreeableness → lower frequency_penalty (comfortable with repetition for emphasis)
- High Neuroticism → slight temperature increase (more emotional variation)

**Drift Detection** (`personalityDriftService.js`):
- Compares recent (7-day) vs baseline (90-day) personality embedding centroids
- Triggers automatic profile rebuild when cosine similarity < 0.85
- Hooked into observation ingestion pipeline (no extra cron job needed)

**Implementation Plan:** `.claude/plans/2026-03-08-soul-signature-voting-layer.md` (10 tasks, 5 phases)

### Synaptic Maturation (CL1-Inspired Neural Memory)
Completes the biological neuron analogy: memories don't just store — they strengthen, decay, and replay like real synapses.

**Three features:**

1. **STDP Exponential Decay** (`cron-memory-forgetting.js` Tier 4) — "Don't fire, connections expire"
   - Co-citation links decay with `new_strength = old * 0.92^max(0, days - 30)`
   - 30-day grace period (recently reinforced links are safe)
   - Links pruned when strength drops below 0.1
   - Runs weekly in the existing memory-forgetting cron
   - `last_reinforced_at` column tracks when links were last co-cited together

2. **Graph-Based Retrieval Traversal** (`memoryLinksService.js` → `memoryStreamService.js`)
   - 1-hop traversal of `memory_links` augments vector search with associatively connected memories
   - Feature-flagged via `graphRetrieval` in `feature_flags` table (default off)
   - Exactly 2 DB queries: batch fetch links from top-5 seed memories, then batch fetch memory rows
   - Injected after MMR reranking, score capped at 80% of top vector result
   - Strength-weighted: stronger links → higher injection scores

3. **Memory Saliency Replay** (`saliencyReplayService.js`) — Neural sleep consolidation
   - Daily cron at 4am UTC replays stale-but-important memories (importance >= 7, not accessed in 14+ days)
   - Refreshes `last_accessed_at = NOW()` to restore recency scores in retrieval
   - Triggers reflection engine for fresh cross-temporal insights connecting old + new memories
   - Cost controls: max 3 users/run, 20 memories/user, respects reflection cooldown
   - Eligible types: `fact`, `platform_data`, `observation` (not reflections — they're already high-level)

**STDP + Graph Retrieval feedback loop:**
```
Memory co-cited in reflection → strengthenCoCitedLinks() → strength ↑ + last_reinforced_at = NOW
                                                              ↓
                                          Graph retrieval picks up strong links
                                                              ↓
                                          Connected memories surface in twin chat
                                                              ↓
                                          More co-citations → strength ↑↑ (Hebbian learning)

No co-citations for 30+ days → STDP decay kicks in → strength ↓↓ → pruned at 0.1
```

**References:**
- Park et al., "Generative Agents" (UIST 2023) — Memory stream architecture
- CL1_LLM_Encoder (Cortical Labs) — Biological neuron blending formula, synaptic plasticity
- arXiv:2412.00804 — Personality drift detection in LLM agents
- STDP (Spike-Timing Dependent Plasticity) — Biological synaptic strengthening/weakening model
- Eon Systems whole-brain emulation (Nature, Oct 2024) — Neurotransmitter distribution informing mode-based parameter modulation

### Neurotransmitter Modes (`neurotransmitterService.js`)
Context-dependent dynamic modulation of sampling parameters. Inspired by how neurotransmitters globally shift brain processing. All functions are PURE (no DB, no LLM, microseconds).

**Three modes** (additive deltas on top of OCEAN-derived personality params):
- **Serotonergic** (emotional/supportive): temp +0.05, freq_pen -0.08, pres_pen +0.05 — warmer, allows comforting repetition
- **Dopaminergic** (analytical/goal-focused): temp -0.08, top_p -0.03, freq_pen +0.08, pres_pen -0.05 — precise, varied vocabulary
- **Noradrenergic** (creative/exploratory): temp +0.10, top_p +0.05, freq_pen +0.03, pres_pen +0.03 — widest sampling

**Detection**: Keyword-based classification, requires >= 2 keyword matches to activate. Returns `{ mode, confidence, matchedKeywords }`.
**Feature flag**: `neurotransmitter_modes` (default: enabled)

### Connectome Neuropils (`neuropilRouter.js`)
Domain-specific memory retrieval routing. Maps 5 brain regions to the 5 reflection expert domains with custom retrieval weights and type budgets.

**Five neuropils**:
- `personality`: identity-like weights [0.3, 0.8, 1.0], more reflections+conversations
- `lifestyle`: recent-biased [1.0, 0.5, 0.8], more platform_data (10)
- `cultural`: balanced [0.5, 0.7, 1.0], standard mix
- `social`: balanced-recent [0.7, 0.6, 1.0], more conversations (8)
- `motivation`: recent+importance [0.8, 0.7, 1.0], more facts (8)

**Routing**: `classifyNeuropil(message)` → `{ neuropilId, weights, budgets, confidence }`. Budgets passed to `retrieveDiverseMemories()`.
**Feature flag**: `connectome_neuropils` (default: enabled)

### Embodied Feedback Loop (Nudges)
Closes the sensorimotor loop: twin suggests micro-action → user acts → twin observes result → learns what works.

**Pipeline**: `generateProactiveInsights()` → 'nudge' category with `nudge_action` → delivered in chat → `evaluateNudgeOutcomes()` (12-48h later, keyword overlap with platform_data) → outcome stored → injected as "PAST NUDGES" context in future chats.

**Functions** (in `proactiveInsights.js`):
- `evaluateNudgeOutcomes(userId)` — keyword overlap scan of platform_data vs nudge_action, ≥40% match = followed
- `getNudgeHistory(userId, limit)` — recent evaluated nudges for chat context
- `getNudgeEffectivenessScore(userId)` — followed/total ratio (last 30d)

**Feature flag**: `embodied_feedback_loop` (default: enabled)
**Migration**: `20260309_add_nudge_feedback_columns` — adds nudge_action, nudge_followed, nudge_outcome, nudge_checked_at to proactive_insights

### Key Architecture Files
- `api/services/memoryStreamService.js` - Write/read path for memory stream (per-utterance storage)
- `api/services/reflectionEngine.js` - Reflection generation pipeline (recursive, depth 3)
- `api/services/twinSummaryService.js` - Dynamic twin summary generation + caching
- `api/services/proactiveInsights.js` - Proactive insight generation + delivery tracking
- `api/services/observationIngestion.js` - Background platform data -> observation pipeline + goal tracking hooks
- `api/services/goalTrackingService.js` - Goal CRUD, suggestion engine, auto-progress tracking, metric extraction
- `api/services/embeddingService.js` - Vector embeddings (text-embedding-3-small, 1536d)
- `api/services/llmGateway.js` - Unified LLM gateway (OpenRouter + caching + cost tracking)
- `api/routes/twin-chat.js` - Twin chat endpoint with full context pipeline
- `api/routes/goals.js` - Goal tracking API endpoints
- `api/config/aiModels.js` - Model tiers, pricing, OpenRouter config
- `api/services/extractionOrchestrator.js` - Platform data extraction coordinator
- `api/services/personalityProfileService.js` - OCEAN extraction, stylometrics, sampling derivation, personality embedding centroid
- `api/services/personalityPromptBuilder.js` - OCEAN-to-prompt-instruction translator for system prompt injection
- `api/services/personalityReranker.js` - Best-of-N reranking with personality embedding cosine similarity
- `api/services/personalityDriftService.js` - Drift detection (7-day vs 90-day) and automatic profile rebuild
- `api/routes/personality-profile.js` - API endpoints (GET profile, POST rebuild, GET drift)
- `api/services/memoryLinksService.js` - Memory graph: auto-linking, co-citation strengthening, graph traversal for retrieval
- `api/services/saliencyReplayService.js` - CL1-inspired sleep consolidation: replay stale important memories
- `api/routes/cron-memory-saliency-replay.js` - Daily 4am cron for saliency replay
- `api/routes/cron-memory-forgetting.js` - Weekly memory quality: soft-delete, STDP decay, link pruning
- `api/services/neurotransmitterService.js` - Context-dependent sampling parameter modulation (3 pure functions)
- `api/services/neuropilRouter.js` - Domain-specific memory retrieval routing (5 neuropils)

## Tech Stack
- **Frontend**: React 18, TypeScript, Vite, Tailwind CSS, Framer Motion, shadcn/ui
- **Backend**: Node.js, Express 5, JWT Auth
- **Database**: Supabase (PostgreSQL + pgvector) - ONLY active database
- **AI**: OpenRouter (DeepSeek V3.2 for analysis, Mistral Small for extraction, Claude Sonnet for twin chat)
- **LLM Gateway**: `api/services/llmGateway.js` - ALL LLM calls route through here
- **Cache**: Redis (ioredis) with in-memory fallback
- **Auth**: JWT + OAuth 2.0 for platform connections
- **Analytics**: PostHog

## Active Platform Integrations (10)
1. **Spotify** - Music taste, listening patterns, mood
2. **Google Calendar** - Schedule, events, time patterns
3. **YouTube** - Content preferences, subscriptions
4. **Gmail** - Communication patterns from email metadata
5. **Discord** - Server activity, community interests, communication style
6. **LinkedIn** - Career trajectory, professional skills, network
7. **GitHub** - Coding activity and open source contributions
8. **Reddit** - Community interests and discussion patterns
9. **Twitch** - Gaming identity and streaming preferences
10. **Whoop** - Recovery, strain, sleep, HRV patterns

## LLM Model Strategy
| Tier | Use Case | OpenRouter Model ID | Why |
|------|----------|---------------------|-----|
| CHAT | Twin conversation | `anthropic/claude-sonnet-4.5` | Quality matters - twin must feel like YOU |
| ANALYSIS | Reflections, twin summary, proactive insights | `deepseek/deepseek-v3.2` | Good enough, 95% cheaper |
| EXTRACTION | Importance rating, fact extraction | `mistralai/mistral-small-creative` | Cheapest, structured output |

## Development
```bash
npm run dev          # Frontend: http://localhost:8086
npm run server:dev   # Backend: http://localhost:3004
npm run dev:full     # Both together
```

## Project Structure
```
twin-ai-learn/
├── src/                    # Frontend (React + TypeScript)
│   ├── pages/              # Route pages
│   ├── components/         # Reusable components
│   ├── contexts/           # React Context providers
│   ├── services/           # API client layer
│   └── hooks/              # Custom hooks
├── api/                    # Backend (Express)
│   ├── routes/             # API endpoints
│   ├── services/           # Business logic + memory architecture
│   ├── middleware/          # Auth, rate limiting, validation
│   └── config/             # AI models, constants
├── database/               # Supabase migrations
└── browser-extension/      # Chrome extension
```

## Environment Variables (Required)
```
NODE_ENV, PORT, VITE_APP_URL, VITE_API_URL
VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY
JWT_SECRET, ENCRYPTION_KEY
OPENROUTER_API_KEY
SPOTIFY_CLIENT_ID, SPOTIFY_CLIENT_SECRET
GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
YOUTUBE_API_KEY
```

## Philosophy
- **From Resume to Soul**: Moving beyond professional achievements to authentic personality
- **Instant Wow**: Users should be surprised by what we know in the first 60 seconds
- **Privacy as Feature**: The privacy spectrum dashboard IS the trust builder
- **Quality over Quantity**: 5 great integrations > 56 half-built ones
- **The Twin Must Have Soul**: Not ChatGPT with facts - it must EMBODY the user's personality
- **Memory Is Everything**: The twin's quality is directly proportional to how well its memory stream works

## Critical Gotchas

### User IDs: public.users NOT auth.users
The app uses `public.users.id` everywhere (user_memories, twin_goals, etc.), NOT `auth.users.id`. These are DIFFERENT UUIDs. All FK constraints reference `public.users(id)`. The test user is `167c27b5-a40b-49fb-8d00-deb1b1c57f4d` (stefanogebara@gmail.com).

### JWT Token Format
Auth middleware reads `payload.id || payload.userId`. The verify endpoint uses `decoded.id`. ALWAYS use `id` field when generating test tokens.

### Frontend API Base URL
`VITE_API_URL=http://127.0.0.1:3004/api` already includes `/api`. Frontend API clients use paths like `/goals` not `/api/goals` to avoid double prefix.

### Memory Stream Composition
Recent memories are dominated by reflections (~90 of last 100). Platform data observations are sparse (~4 in 200). When scanning for platform data, fetch 200+ memories and filter by `memory_type === 'platform_data'`.

### Windows Process Management on Git Bash
`taskkill /PID 12345 /F` fails in Git Bash due to path expansion (`/PID` -> `C:/Program Files/Git/PID`). Use `cmd.exe //c "taskkill /PID 12345 /F"` instead.

## NODE PROCESS MANAGEMENT
**NEVER kill ALL node processes (crashes the CLI):**
- `taskkill /F /IM node.exe` - NEVER
- `pkill node` - NEVER

**OK to kill specific processes by PID:**
- `cmd.exe //c "taskkill /PID 12345 /F"` - OK when you know the specific PID

## Custom Slash Commands
- `/verify-app` - TypeScript check + Vite build + server health
- `/test-api <endpoint>` - Test API endpoints with auth
- `/test-twin <message>` - Test twin chat context pipeline
- `/code-review` - Full code review of current branch
- `/design-review` - Design review with browser testing

---

## Design System (Dark Mode — Claura)
> Dark-only design system. ThemeContext is hard-locked to dark mode (no light mode exists).
> CSS tokens in `src/index.css`, opacity scale in `src/styles/tokens.ts`.

### Color Tokens (`:root` — single dark theme)

**Backgrounds:**
- `--background: #13121a` — night sky base
- `--surface: rgba(255,255,255,0.06)` — elevated surface
- `--surface-solid: rgba(255,255,255,0.12)` — raised surface (hover, active)

**Text:**
- `--foreground: #F5F5F4` — primary text (warm stone-50)
- `--text-primary: #F5F5F4`
- `--text-secondary: #A8A29E` — body secondary (stone-400)
- `--text-muted: #9C9590` — labels, captions
- `--text-placeholder: #57534E` — input placeholders (stone-600)

**Glass Surface (dark glassmorphism):**
- `--glass-surface-bg: rgba(255,255,255,0.06)` — card/panel fill
- `--glass-surface-border: rgba(255,255,255,0.10)` — card border

**Borders:**
- `--border: rgba(255,255,255,0.08)` — default border
- `--border-glass: rgba(255,255,255,0.06)` — subtle glass border

**Interactive:**
- `--primary: #F5F5F4` — primary button bg (light on dark)
- `--primary-foreground: #110f0f` — primary button text (dark)
- `--ring: rgba(255,132,0,0.25)` — focus ring
- `--input: rgba(255,255,255,0.08)` — input field bg

**Component colors:**
- `--card: rgba(255,255,255,0.06)` / `--card-foreground: #F5F5F4`
- `--popover: rgba(40,37,36,0.95)` / `--popover-foreground: #F5F5F4`
- `--secondary: rgba(255,255,255,0.08)` / `--secondary-foreground: #F5F5F4`
- `--muted: rgba(17,15,15,0.8)` / `--muted-foreground: #A8A29E`
- `--accent: rgba(255,255,255,0.06)` / `--accent-foreground: #F5F5F4`
- `--destructive: #dc2626`

**Sidebar:**
- `--sidebar: rgba(255,255,255,0.05)` / `--sidebar-foreground: #D6D3D1`
- `--sidebar-accent: rgba(255,255,255,0.08)` / `--sidebar-border: rgba(255,255,255,0.08)`

**Accent (vibrant):**
- `--accent-vibrant: #ff8400` — orange CTA highlight
- `--accent-vibrant-glow: rgba(255,132,0,0.12)` — active nav pill fill
- `--accent-amber: #c17e2c` — warm copper (gradient core)
- `--accent-purple: #5d5cae` — cool purple (gradient corner only)

**Narrative text (Sesame-inspired opacity hierarchy):**
- `--text-narrative: rgba(245,245,244,0.9)` — primary narrative
- `--text-narrative-secondary: rgba(245,245,244,0.6)` — body narrative
- `--text-narrative-muted: rgba(245,245,244,0.4)` — captions, timestamps

### Background Gradient System (Sun-Driven Ambient Orbs)

The page bg uses FOUR overlapping radial gradients with positions/sizes driven by `SunContext` (time-of-day animation). Default values in `:root`:

```css
body {
  background-color: var(--background);  /* #13121a */
  background-image:
    radial-gradient(ellipse var(--bg-size-1) at var(--bg-pos-1), var(--body-gradient-1) 0%, transparent var(--bg-spread-1)),
    radial-gradient(ellipse var(--bg-size-2) at var(--bg-pos-2), var(--body-gradient-2) 0%, transparent var(--bg-spread-2)),
    radial-gradient(ellipse var(--bg-size-3) at var(--bg-pos-3), var(--body-gradient-3) 0%, transparent var(--bg-spread-3)),
    radial-gradient(ellipse var(--bg-size-4) at var(--bg-pos-4), var(--body-gradient-4) 0%, transparent var(--bg-spread-4));
  background-attachment: fixed;
}
```

**Default gradient colors (amber/copper on charcoal):**
- Orb 1: `rgba(210,145,55,0.38)` — warm amber, top-left
- Orb 2: `rgba(180,110,65,0.30)` — copper, top-right
- Orb 3: `rgba(160,95,55,0.34)` — deep amber, bottom-center
- Orb 4: `rgba(55,45,140,0.28)` — purple accent, center-right

### Glass Surface (REQUIRED for all cards/panels)
```css
background: var(--glass-surface-bg);           /* rgba(255,255,255,0.06) */
backdrop-filter: blur(42px);
-webkit-backdrop-filter: blur(42px);
border: 1px solid var(--glass-surface-border); /* rgba(255,255,255,0.10) */
border-radius: 20px;
box-shadow: 0 4px 4px rgba(0,0,0,0.12), inset 0 1px 0 rgba(255,255,255,0.06);
```

### Blur + Radius Reference
| Element                 | backdrop-filter  | border-radius  | padding              |
|-------------------------|------------------|----------------|----------------------|
| Floating navbar         | blur(19.65px)    | 32px           | pl-5 pr-3 py-2.5     |
| Cards / Chatbox         | blur(42px)       | 20px           | px-5 py-4            |
| Suggestion pills        | blur(42px)       | 46px           | px-3 py-2.5          |
| Auth modal card         | blur(51px)       | 24px           | px-6 py-4            |
| Settings sidebar        | blur(42px)       | 8px            | pt-3 px-5            |
| Top navbar (app)        | blur(16px)       | 0 (full-width) | p-3                  |
| Share button (ghost)    | blur(16px)       | 6px            | px-2 py-0.5          |

### Typography
- **`Instrument Serif`** — hero headings, auth titles, brand name, narrative voice (`.text-heading`, `.narrative-voice`). Weight 400, tracking -0.02em to -0.05em
  - Auth title: `text-[36px] tracking-[-0.72px]` / Hero: `text-[48px] tracking-[-0.96px]` / Large: `text-[56px] tracking-[-1.12px]`
- **`Geist` / `Inter`** — ALL UI text (body, labels, inputs, buttons, nav). Font stack: `'Geist', 'Inter', system-ui, sans-serif`
  - Regular (400): labels, descriptions, placeholders. Weight 500 is the base body weight.
  - Medium (500): body text, button text, active states
  - Semi Bold (600): headings h4-h6, breadcrumbs
- Scale: `text-xs 12px` / `text-sm 14px` / `14.5px base` / `text-base 16px` / `text-lg 18px` / `text-xl 20px`

### Component Specs

**Glass Card** (standard):
- `bg-[rgba(255,255,255,0.06)] border border-[rgba(255,255,255,0.10)] rounded-[20px] px-5 py-4 backdrop-blur-[42px]`

**AI Chatbox** (`backdrop-blur(42px)` card):
- `bg-[rgba(255,255,255,0.06)] border border-[rgba(255,255,255,0.10)] rounded-[20px] px-5 py-4`
- Placeholder: Geist/Inter 14px `#57534E`

**Send Button** (light pill on dark):
- `bg-[#F5F5F4] text-[#110f0f] rounded-[100px] p-[4px]` — total `28x28px` with `20x20` arrow icon
- `opacity-50` when disabled

**Primary CTA Button** (filled pill):
- `bg-[#F5F5F4] text-[#110f0f] rounded-[100px] px-3 py-2 min-w-[80px]`
- Geist/Inter Medium 14px

**Secondary/Ghost Button** (transparent on dark):
- No background, `rounded-[6px] px-2 py-0.5`
- Geist/Inter Medium 12px, `text-[#F5F5F4]`

**Input Field**:
- `bg-[rgba(255,255,255,0.08)] border border-[rgba(255,255,255,0.08)] rounded-[6px]`
- `px-3 py-2.5`
- Label above: 12px `#9C9590` / Placeholder: 14px `#57534E`

**Suggestion Pills** (glass chips):
- `backdrop-blur-[42px] bg-[rgba(255,255,255,0.06)] border border-[rgba(255,255,255,0.10)]`
- `rounded-[46px] px-3 py-2.5 gap-1`
- Icon 16px + Geist/Inter Medium 12px

**Sidebar Nav (app sidebar)**:
- Floating pill `rounded-[32px]` (NOT full-width bar)
- Active item: `bg-[var(--accent-vibrant-glow)] rounded-full px-4 py-2.5`
- Icon: `color: var(--accent-vibrant)` when active
- Inactive: transparent + hover `bg-sidebar-accent`
- Font: Geist/Inter Medium 14px

**Shadow tokens**:
- `translucent`: `blur(84px)` + `drop-shadow(0 4px 4px rgba(0,0,0,0.12))`
- `shadow-lg`: `drop-shadow(0 4px 6px rgba(0,0,0,0.1)) drop-shadow(0 10px 15px rgba(0,0,0,0.1))`

### 12 Rules for AI Code Generation

1. Every card/panel → glass surface (`backdrop-blur(42px)`, `rgba(255,255,255,0.06)` bg, `rgba(255,255,255,0.10)` border)
2. Never flat white or solid colors on app surfaces — always dark glass with white-alpha values
3. Page wrapper → `--background` (#13121a) + 4-orb sun-driven ambient gradient (see SunContext)
4. Primary CTA → `rounded-[100px]` pill, light-on-dark (`#F5F5F4` bg, `#110f0f` text)
5. Suggestion chips → `rounded-[46px]`, NOT `rounded-full`
6. Floating navbar → `rounded-[32px]` pill with `blur(19.65px)`, NOT full-width bar
7. Font → Geist/Inter for ALL UI; Instrument Serif for hero/display/auth titles and narrative voice only
8. Headings → always negative letter-spacing (`tracking-tight` or explicit negative value)
9. Active sidebar nav → full pill fill (`accent-vibrant-glow`), icon in `accent-vibrant`, NEVER underline or left border
10. Gradients → amber/copper orbs on #13121a charcoal, purple accent orb — never neon, never flat
11. Input fields → `rgba(255,255,255,0.08)` bg (NOT glass-surface-bg) — subtler fill
12. ThemeContext is hard-locked to dark. No `.dark` class toggling — all tokens are dark-only in `:root`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stefanogebara)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/stefanogebara)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
