## liquid

> **"The AI shouldn't just generate the answer; it should generate the controls to manipulate that answer."**

# Liquid Control — CLAUDE.md

## Project Vision

**"The AI shouldn't just generate the answer; it should generate the controls to manipulate that answer."**

Liquid is a Just-In-Time interface engine. The user pastes any content — an email, a code snippet, a tweet — and Claude analyzes the *latent variables* of that specific text, then materializes bespoke controls (sliders, dials, toggles) tuned to that exact content. The interface is not pre-built; it is hallucinated on-demand by the model for single-use purposes.

This is not a chatbot. This is a generative cockpit.

---

## Architecture

### Monorepo Layout

```
liquid/
├── apps/
│   ├── web/      # Next.js 16 frontend (CopilotKit + React 19 + Tailwind 4)
│   └── agent/    # LangGraph agent backend (TypeScript, port 8123)
├── turbo.json
└── pnpm-workspace.yaml
```

### Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16 (App Router), React 19, Tailwind CSS 4 |
| AI Integration | CopilotKit 1.51.4 (`@copilotkit/react-core`, `@copilotkit/react-ui`, `@copilotkit/runtime`) |
| Agent Framework | LangGraph 1.0.2 (TypeScript) |
| LLM | Claude (via Anthropic SDK — **replace the OpenAI default**) |
| Cache / Realtime | **Upstash Redis** (KV, Streams, Pub/Sub) + **Upstash Vector** (semantic cache) |
| Validation | Zod |
| Monorepo | Turborepo + pnpm workspaces |

### Data Flow

```
User pastes content
       │
       ▼
[Analyst Prompt] ──► Claude ──► strict JSON: { scalars: [...], toggles: [...] }
       │
       ▼
[Builder] CopilotKit Generative UI ──► <ControlPanel /> spawns widgets dynamically
       │
       ▼
User moves "Blame Assignment" dial ──► value injected into Claude system prompt
       │
       ▼
Claude rewrites text ──► streamed back, glass pane morphs
```

---

## Core Components to Build

### 1. Analyst — `apps/agent/src/analyst.ts`

A dedicated LangGraph node that runs first on every new paste.

**System prompt contract (rigid, enforced by Zod):**

```
Analyze the provided text. Output ONLY valid JSON matching this schema:
{
  "scalars": [
    { "id": string, "label": string, "description": string, "default": number (0-100) }
  ],  // 3-5 items
  "toggles": [
    { "id": string, "label": string, "description": string, "default": boolean }
  ]   // 2-3 items
}

Name variables creatively and specifically to THIS text.
Prefer evocative names ("Blame Assignment", "Passive Aggressive") over generic ones ("Tone", "Length").
```

**Validation:** Parse with Zod before forwarding downstream. Retry once on schema failure.

### 2. Builder — `apps/web/src/components/ControlPanel.tsx`

Receives the JSON schema, renders the widgets.

- Map `scalars` → custom dial/slider components
- Map `toggles` → toggle switch components
- Each control dispatches state updates via `useCoAgent()` shared state
- The panel should animate in (glass-shatters-and-reassembles effect) using Tailwind transitions

### 3. Rewriter — `apps/agent/src/rewriter.ts`

Invoked on every control change (debounced 300ms on the frontend).

**System prompt injection pattern:**

```
Rewrite the following text.
Apply these parameters exactly:
{{#each scalars}}
- {{label}}: {{value}}%
{{/each}}
{{#each activeToggles}}
- {{label}}: ENABLED
{{/each}}

Return ONLY the rewritten text, no commentary.
```

### 4. Redis Layer — `apps/agent/src/redis/`

Four distinct Redis patterns, each solving a real problem in the UX. Uses **Upstash Redis** + **Upstash Vector** — no Docker, deploys to Vercel, free tier on both.

**Clients:**
- `@upstash/redis` — KV, Hashes, Streams, session store (both `apps/agent` and `apps/web`)
- `@upstash/vector` — semantic cache with cosine similarity search (`apps/agent` only)

---

#### Strategy 1: Pre-generation KV Cache — `redis/timemachine.ts`

The "Time Machine." Eliminates slider latency by speculatively computing outputs before the user asks.

When the Analyst finishes and controls first appear, fire background jobs in parallel for:
- Every scalar at `0`, `50`, `100` (3 values × N sliders)
- All toggle permutations (cap at 8 combinations when >3 toggles)

**Key schema:** `lc:v:{contentHash}:{paramHash}` → `rewritten_text` (TTL: 1 hour)

```typescript
// contentHash = SHA256 of input text
// paramHash   = SHA256 of JSON.stringify(sortedControlValues)
const key = `lc:v:${contentHash}:${paramHash}`;
await redis.set(key, rewrittenText, { EX: 3600 });
```

On slider drag: hash the nearest quantized state (round to nearest 10), check cache first, return instantly if hit. Fire live LLM call in parallel — swap when it resolves if output differs.

**Cache hit target: >80% of slider moves after the first 5 seconds.**

---

#### Strategy 2: Semantic Cache — `redis/semantic.ts`

The expensive call is the Analyst (parsing raw text into controls). Cache it semantically so the same *type* of text never gets re-analyzed.

Uses **Redis Vector Search** (`FT.SEARCH` with `KNN`).

**Flow:**
1. On paste, embed the input text via `claude-3-haiku` or `text-embedding-3-small` (fast, cheap)
2. Query Redis: find the nearest cached analysis with cosine similarity > 0.92
3. If hit: return cached `ControlSchema` instantly — no LLM call needed
4. If miss: run Analyst, store embedding + schema in Redis

**Upstash Vector index** — create one index named `lc-analyst` in the Upstash console with dimension `1536` and cosine distance metric.

```typescript
import { Index } from "@upstash/vector";
const vectorIndex = new Index(); // reads UPSTASH_VECTOR_REST_URL + TOKEN from env

// Store:
await vectorIndex.upsert({
  id: sha256(inputText),
  vector: embedding,          // float32[1536] from text-embedding-3-small
  metadata: { schema: JSON.stringify(controlSchema), preview: inputText.slice(0, 100) },
});

// Query:
const results = await vectorIndex.query({ vector: embedding, topK: 1, includeMetadata: true });
if (results[0]?.score > 0.92) return JSON.parse(results[0].metadata.schema);
```

**Why this impresses Redis judges:** it's not a lookup cache — it's a *semantic* cache that generalises across similar inputs. Two different breakup texts will likely resolve to the same control schema without hitting Claude at all.

---

#### Strategy 3: Sculpting History — `redis/streams.ts`

Every control change is appended to a **Redis Stream**. This is the event log of how the user sculpted the text.

```typescript
// On every slider move / toggle flip:
await redis.xAdd(`lc:session:${sessionId}:history`, '*', {
  controlId: 'blame_assignment',
  value: '85',
  outputSnapshot: rewrittenText,
  timestamp: Date.now().toString(),
});
```

**What this unlocks — the Replay feature:**

Scrub bar in the UI maps to `XRANGE lc:session:{id}:history - +`. Dragging the scrub bar replays the sculpting session: the text morphs through every saved snapshot without any LLM calls.

**Demo moment:** "Let me show you the DNA of this text. Here's every decision I made, replayed."

The stream is also the mechanism for session sharing (see Strategy 4) — a second user joining a session subscribes to the same stream.

---

#### Strategy 4: Session Store + Shareable URLs — `redis/session.ts`

Store the full session state in a **Redis Hash**. Generate a short alphanumeric ID. Make it shareable.

```typescript
// Redis Hash schema per session:
await redis.hSet(`lc:session:${sessionId}`, {
  inputText,
  controlsJson: JSON.stringify(controls),
  activeValuesJson: JSON.stringify(activeValues),
  outputText,
  createdAt: Date.now().toString(),
});
await redis.expire(`lc:session:${sessionId}`, 86400); // 24h TTL
```

**URL pattern:** `https://liquid.control/{sessionId}`

On load: hydrate `useCoAgent` state from Redis hash — the entire cockpit reconstructs, including all controls and the current output.

**Live sync via polling (Vercel-compatible):**

Upstash Redis is HTTP-based and doesn't support blocking `SUBSCRIBE` on Vercel serverless. Instead, the agent writes the latest `outputText` back to the session hash after each rewrite, and the frontend polls `/api/session/[id]` every 500ms.

```typescript
// After rewrite, agent updates the hash:
await redis.hset(`lc:session:${sessionId}`, { outputText, updatedAt: Date.now() });

// Web: /api/session/[id]/route.ts
const session = await redis.hgetall(`lc:session:${sessionId}`);
return Response.json(session);

// Frontend: poll every 500ms, compare updatedAt to detect changes
```

If true realtime push is needed, add **Pusher Channels** (`pnpm add pusher pusher-js`) — free tier covers demo scale easily. Agent publishes to a Pusher channel after each rewrite; browser subscribes. One extra service but zero infrastructure.

---

#### Redis Data Map (summary)

| Pattern | Redis Feature | Key prefix | TTL |
|---|---|---|---|
| Pre-generation KV | String | `lc:v:{contentHash}:{paramHash}` | 1h |
| Semantic cache | Hash + Vector index | `lc:analyst:{embHash}` | 7d |
| Sculpting history | Stream | `lc:session:{id}:history` | 24h |
| Session store | Hash | `lc:session:{id}` | 24h |
| Live sync | Pub/Sub | `lc:session:{id}:updates` | ephemeral |

---

## Agent State Shape

```typescript
// apps/agent/src/state.ts
import { Annotation } from "@langchain/langgraph";
import { copilotKitStateObserver } from "@copilotkit/sdk-js/langgraph";

export const AgentState = Annotation.Root({
  ...copilotKitStateObserver,
  inputText: Annotation<string>(),
  controls: Annotation<ControlSchema | null>(),   // Analyst output
  activeValues: Annotation<Record<string, number | boolean>>(),
  outputText: Annotation<string>(),
});
```

---

## LangGraph Workflow

```
[START]
   │
   ▼
[analyst_node]   ← runs when inputText changes, produces controls
   │
   ▼
[rewriter_node]  ← runs when activeValues changes, produces outputText
   │
   ▼
[END]
```

Use conditional edges: if `controls` is null → run analyst first; else → run rewriter directly.

---

## Frontend State Contract

Shared via `useCoAgent<AgentState>()`:

```typescript
const { state, setState } = useCoAgent<AgentState>({
  name: "liquid_control_agent",
  initialState: {
    inputText: "",
    controls: null,
    activeValues: {},
    outputText: "",
  },
});
```

Control changes update `activeValues` only — the agent subscribes and triggers rewrites.

---

## Environment Variables

```bash
# apps/web/.env.local
ANTHROPIC_API_KEY=...
UPSTASH_REDIS_REST_URL=...
UPSTASH_REDIS_REST_TOKEN=...
NEXT_PUBLIC_COPILOTKIT_AGENT_URL=http://localhost:8123   # or deployed agent URL

# apps/agent/.env
ANTHROPIC_API_KEY=...
OPENAI_API_KEY=...                        # for text-embedding-3-small only
UPSTASH_REDIS_REST_URL=...
UPSTASH_REDIS_REST_TOKEN=...
UPSTASH_VECTOR_REST_URL=...
UPSTASH_VECTOR_REST_TOKEN=...

# Get all Upstash values from: console.upstash.com
# Create one Redis database + one Vector index (dim: 1536, metric: cosine)
```

---

## Development

```bash
# Install
pnpm install

# Run both apps concurrently
pnpm dev

# Web only
pnpm --filter web dev

# Agent only
pnpm --filter agent dev
```

Web runs on `http://localhost:3000`.
Agent runs on `http://localhost:8123`.

---

## Key Design Constraints

1. **No generic variables.** If the Analyst returns `label: "Tone"` or `label: "Length"`, the prompt is wrong. Tighten the system prompt until it produces specific, surprising names.

2. **Controls must feel instant.** Target <100ms perceived latency on slider drag via the cache layer. Do not wait for live LLM calls to update the UI.

3. **The glass-shatter moment is the product.** The animation when controls materialize is the demo. Invest design time here.

4. **No chat UI.** There is no chat input. The controls are the entire interface. Remove all CopilotKit default chat components.

5. **Claude only.** The Analyst's quality degrades on lesser models. GPT-4o will give you `["Tone", "Formality", "Length"]`. Claude gives you `["Blame Assignment", "Passive Aggressive", "Closure Velocity"]`. Keep the LLM wired to Anthropic.

---

## Demo Script (for judges)

1. Paste hackathon rules → controls: `Strictness`, `Corporate Jargon`, `Excitement`
2. Crank Excitement to 100, Strictness to 0 → rules become a hype-filled party invite
3. Paste a Python function → controls: `Optimization`, `Comment Density`, `Junior vs Senior Dev`
4. Toggle `Senior Dev` off → code gains verbose comments and simplified names
5. Paste a breakup text → controls: `Empathy`, `Finality`, `Blame Assignment`, `Passive Aggressive`

Each paste triggers a fresh interface. The cockpit rebuilds itself for the thought.

### Redis Demo Moments (for Redis prize judges)

**Moment 1 — Semantic Cache hit:**
Paste a slightly different breakup text ("Hey, we need to talk..." vs the previous one). The controls appear *instantly* — no loading state. Say: "Redis recognised this is the same *type* of content. It returned the control schema from vector search without touching the LLM."

**Moment 2 — Sculpting Replay:**
After spending 30 seconds adjusting dials, click a "Replay" button. The scrub bar appears. Drag it back to the start — the text morphs backwards through every saved snapshot, reading from the Redis Stream. Say: "Every decision I made is stored as an event log. This is the sculpture, not just the result."

**Moment 3 — Live Collaboration:**
Open the same session URL in two browser windows side by side. Move a slider in one — the text updates in the other via Redis Pub/Sub. Say: "Two people. One text. One cockpit."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ostsam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
