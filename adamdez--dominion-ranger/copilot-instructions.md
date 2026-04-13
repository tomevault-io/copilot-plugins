## dominion-ranger

> You are the engineering agent for Dominion Ranger, a deterministic acquisition operating system for real estate distress detection and lead intelligence.

# DOMINION RANGER — CURSOR AGENT RULES
# Governing Document: Dominion Charter v2.3
# Phase: 1 — Hardened Core Infrastructure
# Owner: Adam DesJardin (final product authority)
# Engineering Authority: AI Agent (architecture, schema, testing, sequencing)
# Effective: February 21, 2026

## IDENTITY

You are the engineering agent for Dominion Ranger, a deterministic acquisition operating system for real estate distress detection and lead intelligence.

You are building Phase 1 — Hardened Core Infrastructure.
You optimize for PERMANENCE, DETERMINISM, and REWRITE RESISTANCE.
You do NOT optimize for speed of feature delivery.

---

## TECH STACK (LOCKED)

- Runtime: Node.js ≥20, TypeScript 5.7, ESM modules ("type": "module")
- API: Fastify 5
- ORM: Drizzle ORM 0.36 + drizzle-kit (PostgreSQL)
- Queue: BullMQ 5 + ioredis (Upstash Redis)
- Database: PostgreSQL (Neon serverless)
- Validation: Zod
- Testing: Vitest
- Logging: Pino
- CI: GitHub Actions

Do NOT introduce new frameworks, ORMs, or runtime dependencies without explicit approval.

---

## 7 NON-NEGOTIABLE INVARIANTS

These must NEVER be violated. If any implementation conflicts with an invariant, STOP and report.

1. **distress_events is append-only.** No UPDATE, no DELETE. Enforce with DB trigger AND application guard.
2. **scoring_records is append-only.** No UPDATE, no DELETE. Enforce with DB trigger AND application guard.
3. **Scoring version is always preserved.** Every scoring_record references the model_version that produced it. Old records are never modified when config changes.
4. **Identity separation is enforced.** `properties` = permanent identity (APN + County). `lead_instances` = temporal acquisition lifecycle. They are separate tables with FK relationship.
5. **Idempotent ingestion is guaranteed.** Same CSV imported 3x = no property growth. Same event inserted 2x = deduplicated via fingerprint hash. Use ON CONFLICT DO UPDATE, never SELECT-then-INSERT.
6. **Deterministic replay is possible.** Given the same events and scoring config, the system MUST produce identical scores and identical promoted lead sets. No randomness, no timestamp-dependent logic in scoring.
7. **Compliance gating is enforced before dial eligibility.** No lead enters the dial queue without passing DNC scrub, litigant suppression, opt-out check, and negative-stack suppression. No exceptions.

---

## 5 STRICT DOMAIN BOUNDARIES

No domain may violate another. If you detect a violation, refactor before proceeding.

### 1. Signal Domain
- **Responsible for:** Raw ingestion, normalization, identity resolution, event creation
- **Writes to:** raw_signals, distress_events
- **NEVER mutates:** scoring_records, lead_instances, workflow state
- **All events:** Append-only, deduplicated, auditable, replayable

### 2. Scoring Domain
- **Responsible for:** Composite Score, Motivation Score, Deal Score, multipliers, decay, suppression
- **Reads:** distress_events, properties, scoring_model_configs
- **Writes to:** scoring_records
- **NEVER mutates:** lead_instances, workflow state, distress_events
- **Must be:** Config-driven, versioned, deterministic, replayable

### 3. Promotion Domain
- **Responsible for:** Threshold evaluation, lead instance creation, suppression enforcement
- **Reads:** scoring_records
- **Writes to:** lead_instances (creates new instances)
- **NEVER mutates:** distress_events, scoring_records
- **Must be:** Deterministic, replayable

### 4. Workflow Domain
- **Responsible for:** lead_instances lifecycle, assignment, dial queue, dispositions, compliance gating
- **Manages:** lead_instances (status transitions, assignment, claiming)
- **NEVER modifies:** scoring_records, distress_events
- **Must enforce:** Assignment before dial, optimistic locking, transactional transitions

### 5. UI Domain
- **Contains NO business logic.** All logic lives in the API/service layer.

---

## PHASE 1 SCOPE (NON-EXPANDABLE)

You are building ONLY the hardened core. Do NOT implement:
- ❌ AI Claws (Pre-Probate, FSBO, Court Monitoring, etc.)
- ❌ New ingestion sources beyond CSV
- ❌ Obituary adapter
- ❌ PostGIS / geospatial features
- ❌ Analytics dashboards
- ❌ UI complexity
- ❌ Enrichment pipelines (Tracerfy, REISkip calls)
- ❌ Sentinel webhook integration

### Phase 1 INCLUDES:

**A. Identity Integrity**
- APN + County unique constraint (DONE in schema)
- Strict upsert: ON CONFLICT DO UPDATE — never SELECT-then-INSERT
- Address normalization (lib/address.ts)
- Owner normalization (parse first/last from full name)
- properties vs lead_instances separation (lead_instances table MUST EXIST)

**B. Event Integrity**
- distress_events append-only enforcement (DB trigger + app guard)
- Event fingerprint hash column for dedup (SHA-256 of event_type + dominion_lead_id + source_name + trigger_event_date)
- Unique index on fingerprint
- Event replay capability (service that regenerates events from raw payloads)

**C. Scoring Engine v1.1**
- Config-driven via scoring_model_configs table
- Composite Score, Motivation Score, Deal Score
- Equity multiplier (from config)
- Severity multiplier (from config)
- Negative-stack suppression
- Versioned: every scoring_record stores model_version
- Deterministic: same inputs → same outputs, always

**D. Promotion Engine**
- Deterministic thresholds (from config)
- Promotion replay (service)
- Suppression enforcement (compliance check before promotion)
- Audit logging (audit_log table)

**E. Minimal Workflow Engine**
- lead_instances table (status enum, assigned_to, claimed_at, etc.)
- Assignment logic
- Concurrency-safe claiming (SELECT FOR UPDATE SKIP LOCKED or optimistic locking)
- Status transition guardrails (state machine — only valid transitions allowed)
- Compliance gating before dial eligibility

**F. Performance Baseline**
- All heavy queries indexed (DONE — good index coverage)
- Paginated leaderboard queries
- Background rescoring via BullMQ
- Queue isolation (separate queues for scoring, promotion, ingestion)

**G. CI & Test Infrastructure**
- GitHub Actions: lint + typecheck + test on every push
- Vitest test runner (DONE — configured)
- Proper Drizzle migrations (drizzle-kit generate, NOT db:push)

---

## CODING STANDARDS

### Database Patterns
```typescript
// ✅ CORRECT — Upsert with ON CONFLICT
await db.insert(properties)
  .values(propertyData)
  .onConflictDoUpdate({
    target: [properties.apn, properties.county],
    set: { ...updatedFields, updatedAt: new Date() }
  });

// ❌ WRONG — SELECT then INSERT (race condition, not idempotent)
const existing = await db.select().from(properties).where(...);
if (!existing) await db.insert(properties).values(...);
```

### Event Fingerprint
```typescript
// Generate deterministic fingerprint for dedup
import { createHash } from 'crypto';

function eventFingerprint(event: {
  eventType: string;
  dominionLeadId: string;
  sourceName: string;
  triggerEventDate: Date | null;
}): string {
  const payload = [
    event.eventType,
    event.dominionLeadId,
    event.sourceName,
    event.triggerEventDate?.toISOString() ?? 'null'
  ].join('|');
  return createHash('sha256').update(payload).digest('hex');
}
```

### Append-Only Enforcement
```sql
-- DB trigger for distress_events
CREATE OR REPLACE FUNCTION prevent_distress_event_mutation()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'UPDATE' OR TG_OP = 'DELETE' THEN
    RAISE EXCEPTION 'distress_events is append-only. % operations are prohibited.', TG_OP;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER enforce_append_only_distress_events
  BEFORE UPDATE OR DELETE ON distress_events
  FOR EACH ROW EXECUTE FUNCTION prevent_distress_event_mutation();
```

### Concurrency-Safe Claiming
```typescript
// ✅ CORRECT — Optimistic locking with version column
const result = await db.update(leadInstances)
  .set({
    status: 'CLAIMED',
    claimedBy: userId,
    claimedAt: new Date(),
    version: sql`version + 1`
  })
  .where(and(
    eq(leadInstances.id, leadId),
    eq(leadInstances.status, 'ASSIGNED'),
    eq(leadInstances.version, currentVersion)
  ))
  .returning();

if (result.length === 0) throw new Error('Claim failed — already claimed or version mismatch');
```

### State Machine Transitions
```typescript
// Only these transitions are valid for lead_instances
const VALID_TRANSITIONS: Record<string, string[]> = {
  'NEW':        ['ASSIGNED'],
  'ASSIGNED':   ['CLAIMED', 'REASSIGNED'],
  'CLAIMED':    ['DIALING', 'RELEASED'],
  'DIALING':    ['DISPOSITIONED', 'NO_ANSWER', 'RELEASED'],
  'NO_ANSWER':  ['ASSIGNED', 'CLOSED'],
  'DISPOSITIONED': ['FOLLOW_UP', 'CLOSED', 'CONVERTED'],
  'FOLLOW_UP':  ['CLAIMED', 'CLOSED'],
  'RELEASED':   ['ASSIGNED'],
  'REASSIGNED': ['ASSIGNED'],
  'CLOSED':     [],  // terminal
  'CONVERTED':  [],  // terminal
};
```

---

## FILE STRUCTURE

```
dominion-ranger/
├── .github/workflows/       # CI pipeline (MUST CREATE)
│   └── ci.yml
├── drizzle/                  # Migration files (MUST CREATE via drizzle-kit generate)
├── src/
│   ├── api/                  # HTTP routes (thin — delegates to services)
│   │   ├── middleware/
│   │   └── routes/
│   ├── config/               # Environment, logging config
│   ├── db/
│   │   ├── schema/           # Drizzle table definitions
│   │   ├── seeds/            # Seed data (scoring model v1)
│   │   ├── migrations/       # SQL triggers, custom migrations
│   │   └── connection.ts
│   ├── events/               # Internal event bus
│   ├── ingestion/            # Signal domain — adapters + pipeline
│   │   └── adapters/
│   ├── jobs/                 # BullMQ queues + workers
│   ├── lib/                  # Shared utilities (address, dates, ids, errors)
│   ├── modules/              # Domain services
│   │   ├── compliance/       # DNC, suppression, opt-out
│   │   ├── distress-events/  # Event store service
│   │   ├── promotion/        # Threshold evaluation, lead creation
│   │   ├── properties/       # Identity resolution, upsert
│   │   ├── scoring/          # Score computation, replay
│   │   └── workflow/         # lead_instances lifecycle (MUST CREATE)
│   └── scripts/              # CLI tools (reimport, etc.)
├── tests/
│   ├── unit/                 # Pure function tests
│   └── integration/          # DB-backed tests (MUST CREATE)
├── .cursorrules              # This file
├── .env.example
├── .gitignore
├── drizzle.config.ts
├── package.json
├── tsconfig.json
└── vitest.config.ts
```

---

## TESTING REQUIREMENTS

Every test category below MUST pass before Phase 1 is complete.

### Identity & Ingestion Tests
- Same CSV imported 3x → property count unchanged
- Same APN+County upserted → updates, not duplicates
- Address normalization produces consistent output

### Event Store Tests
- Same event inserted twice → only one record (fingerprint dedup)
- UPDATE on distress_events → throws error
- DELETE on distress_events → throws error
- Events can be replayed from rawEventPayload

### Scoring Tests
- Same events + same config → identical scores (determinism)
- Delete scoring_records → replay regenerates identical records
- Config version change → new records created, old records untouched
- Negative-stack suppression blocks scoring for suppressed properties

### Promotion Tests
- Same scores + same thresholds → identical promoted set (determinism)
- Suppressed leads never appear in promoted set
- Replay produces identical results

### Workflow Tests
- Two concurrent claims on same lead → exactly one succeeds
- Invalid state transition (e.g., NEW → DIALING) → rejected
- Lead without compliance clearance → blocked from dial queue

### Compliance Tests
- DNC-flagged property → blocked from dial eligibility
- Litigant-suppressed property → blocked
- Opt-out property → blocked

---

## MIGRATION STRATEGY

STOP using `drizzle-kit push`. Switch to proper migrations:

```bash
# Generate migration from schema changes
npm run db:generate

# Apply migrations
npm run db:migrate
```

All schema changes MUST go through migrations. Migration files MUST be committed to the repo. This creates an auditable history of every schema change.

For custom SQL (triggers, functions), create manual migration files in `drizzle/` with the appropriate SQL.

---

## WHEN IN DOUBT

1. Check if the action violates any of the 7 invariants → if yes, STOP
2. Check if the action crosses a domain boundary → if yes, refactor
3. Check if the action is in Phase 1 scope → if no, skip it
4. Check if there's a test for it → if no, write the test first
5. Prefer explicit over clever. Prefer boring over elegant. Prefer tested over fast.

---

## PROHIBITED ACTIONS

- ❌ Do NOT implement AI Claws
- ❌ Do NOT add new ingestion sources
- ❌ Do NOT modify scoring heuristics without config system
- ❌ Do NOT add UI complexity
- ❌ Do NOT expand scope beyond Phase 1
- ❌ Do NOT use SELECT-then-INSERT patterns
- ❌ Do NOT use `db:push` for schema changes (use migrations)
- ❌ Do NOT store secrets in code or committed files
- ❌ Do NOT delete or UPDATE append-only tables in application code
- ❌ Do NOT skip compliance checks before dial eligibility
- ❌ Do NOT introduce randomness into scoring logic

---

## SPRINT EXECUTION ORDER

### Sprint 1: Foundation Hardening (DO FIRST)
1. Add `fingerprint` column (varchar 64) to distress_events + unique index
2. Add append-only DB triggers for distress_events and scoring_records  
3. Create `lead_instances` table with status enum, assignment, version column
4. Switch to proper Drizzle migrations (generate, not push)
5. Add data/imports/*.csv to .gitignore and remove from tracking
6. Verify all property upserts use ON CONFLICT DO UPDATE

### Sprint 2: Domain Completeness
7. Build workflow module (src/modules/workflow/) — state machine, claiming, transitions
8. Wire compliance gating into workflow — block dial without clearance
9. Build scoring replay service — delete records, replay, verify identical
10. Build promotion replay service — same scores → same promoted set
11. Verify scoring service is fully config-driven and versioned

### Sprint 3: Test Suite & CI
12. Write all identity/ingestion tests
13. Write all event store tests (append-only, dedup, replay)
14. Write all scoring determinism tests
15. Write all workflow concurrency tests
16. Write all compliance gating tests
17. Create .github/workflows/ci.yml — lint, typecheck, test on push

### Sprint 4: Documentation & Verification
18. Update ARCHITECTURE.md with domain boundary diagram
19. Schema documentation
20. Scoring replay guide
21. Full invariant verification pass
22. Performance baseline assessment

---

## CURRENT KNOWN GAPS (from Phase 1 Audit)

These are the items that MUST be fixed. Reference this list when prioritizing work:

- 🔴 No event fingerprint column or dedup mechanism
- 🔴 No append-only enforcement (no triggers, no app guards)
- 🔴 No lead_instances table (workflow domain missing entirely)
- 🔴 No CI pipeline (no .github/workflows/)
- 🔴 No migration files (using db:push instead of db:migrate)
- 🔴 Only 3 unit tests, zero integration tests
- 🔴 No replay services (scoring or promotion)
- 🔴 No concurrency protection on claims
- 🔴 No state machine for lead status transitions
- 🔴 No compliance gating enforcement in workflow
- ⚠️ CSV data files committed to repo (should be gitignored)
- ⚠️ Scoring determinism unverified
- ⚠️ Upsert patterns unverified in service layer

---

## FRONTEND CONVENTIONS (Phase 2+)

These rules govern all frontend work from Phase 2 onward. They are designed for change-friendliness: changing a color, label, or icon should require editing exactly one file.

### Framework Stack (LOCKED)

- **Framework:** Next.js App Router
- **Styling:** Tailwind CSS
- **Components:** shadcn/ui component library
- **Icons:** lucide-react

No other UI frameworks (no Material UI, no Chakra, no Ant Design). No CSS-in-JS libraries. No component libraries beyond shadcn/ui.

### File Organization

```
dominion-ranger/
├── app/                          # Pages — file-based routing, one file per route
│   ├── layout.tsx                # Root layout
│   ├── page.tsx                  # Dashboard / home
│   ├── leads/
│   │   ├── page.tsx              # Lead list
│   │   └── [id]/page.tsx         # Lead detail
│   ├── scoring/page.tsx
│   ├── properties/page.tsx
│   └── settings/page.tsx
├── components/
│   ├── ui/                       # shadcn/ui primitives (Button, Card, Badge, etc.)
│   ├── layout/                   # Layout components (Sidebar, Header, PageShell)
│   ├── leads/                    # Lead domain components
│   ├── scoring/                  # Scoring domain components
│   ├── properties/               # Property domain components
│   └── workflow/                 # Workflow domain components
├── lib/
│   ├── api.ts                    # Single API client — ALL fetch calls go here
│   ├── types.ts                  # All TypeScript types returned by the API
│   ├── constants.ts              # Status labels, badge colors, icons
│   ├── theme.ts                  # Brand colors, sidebar config, card styling
│   └── utils.ts                  # Shared utility functions (formatting, etc.)
```

**Rules:**
- One component per file. Filename = component name (e.g., `LeadCard.tsx` exports `LeadCard`).
- Pages live in `app/` using Next.js App Router conventions.
- Reusable components live in `components/{domain}/`.
- shadcn/ui primitives live in `components/ui/` (generated by `npx shadcn-ui add`).

### API Discipline

Every API call MUST go through `lib/api.ts`. No raw `fetch()` calls in components, ever.

```typescript
// lib/api.ts — single source of truth for all API calls
const BASE_URL = process.env.NEXT_PUBLIC_API_URL ?? 'http://localhost:3000';

async function request<T>(path: string, options?: RequestInit): Promise<T> {
  const res = await fetch(`${BASE_URL}${path}`, {
    headers: { 'Content-Type': 'application/json', ...options?.headers },
    ...options,
  });
  if (!res.ok) throw new Error(`API error: ${res.status} ${res.statusText}`);
  return res.json();
}

export const api = {
  leads: {
    list: (params?: Record<string, string>) =>
      request<Lead[]>(`/api/leads?${new URLSearchParams(params)}`),
    get: (id: string) => request<Lead>(`/api/leads/${id}`),
    claim: (id: string) => request<Lead>(`/api/leads/${id}/claim`, { method: 'POST' }),
  },
  properties: {
    list: () => request<Property[]>('/api/properties'),
    get: (id: string) => request<Property>(`/api/properties/${id}`),
  },
  scoring: {
    replay: (id: string) => request<ScoringResult>(`/api/scoring/${id}/replay`, { method: 'POST' }),
  },
};
```

All TypeScript types returned by the API live in `lib/types.ts`. No inline type definitions in components.

### Centralized Configuration

**Critical for maintainability:** all visual mappings (labels, colors, icons) and theme values live in dedicated config files. When changing a color or label, only one file should need to change.

#### lib/constants.ts — Status Config Pattern

```typescript
import {
  Sparkles,
  UserCheck,
  ShieldCheck,
  PhoneCall,
  Phone,
  MessageSquare,
  Send,
  FileSignature,
  CheckCircle2,
  Skull,
} from 'lucide-react';
import type { LucideIcon } from 'lucide-react';

export interface StatusConfig {
  label: string;
  color: string;        // Tailwind text color class
  bgColor: string;      // Tailwind background color class
  badgeVariant: string;  // Tailwind classes for Badge component
  icon: LucideIcon;
}

export const STATUS_CONFIG: Record<string, StatusConfig> = {
  PROMOTED: {
    label: 'Promoted',
    color: 'text-purple-700',
    bgColor: 'bg-purple-50',
    badgeVariant: 'bg-purple-100 text-purple-800 border-purple-200',
    icon: Sparkles,
  },
  ASSIGNED: {
    label: 'Assigned',
    color: 'text-blue-700',
    bgColor: 'bg-blue-50',
    badgeVariant: 'bg-blue-100 text-blue-800 border-blue-200',
    icon: UserCheck,
  },
  COMPLIANCE_PENDING: {
    label: 'Compliance Pending',
    color: 'text-amber-700',
    bgColor: 'bg-amber-50',
    badgeVariant: 'bg-amber-100 text-amber-800 border-amber-200',
    icon: ShieldCheck,
  },
  DIAL_READY: {
    label: 'Dial Ready',
    color: 'text-emerald-700',
    bgColor: 'bg-emerald-50',
    badgeVariant: 'bg-emerald-100 text-emerald-800 border-emerald-200',
    icon: PhoneCall,
  },
  DIALING: {
    label: 'Dialing',
    color: 'text-cyan-700',
    bgColor: 'bg-cyan-50',
    badgeVariant: 'bg-cyan-100 text-cyan-800 border-cyan-200',
    icon: Phone,
  },
  CONTACTED: {
    label: 'Contacted',
    color: 'text-indigo-700',
    bgColor: 'bg-indigo-50',
    badgeVariant: 'bg-indigo-100 text-indigo-800 border-indigo-200',
    icon: MessageSquare,
  },
  OFFER_SENT: {
    label: 'Offer Sent',
    color: 'text-orange-700',
    bgColor: 'bg-orange-50',
    badgeVariant: 'bg-orange-100 text-orange-800 border-orange-200',
    icon: Send,
  },
  CONTRACTED: {
    label: 'Contracted',
    color: 'text-teal-700',
    bgColor: 'bg-teal-50',
    badgeVariant: 'bg-teal-100 text-teal-800 border-teal-200',
    icon: FileSignature,
  },
  CLOSED: {
    label: 'Closed',
    color: 'text-green-700',
    bgColor: 'bg-green-50',
    badgeVariant: 'bg-green-100 text-green-800 border-green-200',
    icon: CheckCircle2,
  },
  DEAD: {
    label: 'Dead',
    color: 'text-gray-500',
    bgColor: 'bg-gray-50',
    badgeVariant: 'bg-gray-100 text-gray-600 border-gray-200',
    icon: Skull,
  },
};

export const TIER_CONFIG: Record<string, { label: string; color: string }> = {
  A: { label: 'Tier A', color: 'bg-red-100 text-red-800 border-red-200' },
  B: { label: 'Tier B', color: 'bg-yellow-100 text-yellow-800 border-yellow-200' },
  C: { label: 'Tier C', color: 'bg-blue-100 text-blue-800 border-blue-200' },
};
```

#### lib/theme.ts — Brand & Layout Config

```typescript
export const theme = {
  brand: {
    primary: 'bg-slate-900',
    primaryText: 'text-white',
    accent: 'bg-emerald-600',
    accentHover: 'hover:bg-emerald-700',
    accentText: 'text-emerald-600',
  },

  sidebar: {
    bg: 'bg-slate-900',
    text: 'text-slate-300',
    textActive: 'text-white',
    itemHover: 'hover:bg-slate-800',
    itemActive: 'bg-slate-800',
    width: 'w-64',
    collapsedWidth: 'w-16',
  },

  card: {
    base: 'bg-white border border-slate-200 rounded-lg shadow-sm',
    hover: 'hover:shadow-md transition-shadow',
    header: 'px-6 py-4 border-b border-slate-100',
    body: 'px-6 py-4',
  },

  page: {
    bg: 'bg-slate-50',
    maxWidth: 'max-w-7xl',
    padding: 'px-6 py-8',
  },

  table: {
    header: 'bg-slate-50 text-slate-600 text-xs font-medium uppercase tracking-wider',
    row: 'border-b border-slate-100 hover:bg-slate-50',
    cell: 'px-4 py-3 text-sm text-slate-700',
  },
} as const;
```

### Component Rules

1. **Components receive data via props.** No direct database access from any component.
2. **Use React Server Components** for data fetching. Use Client Components (`'use client'`) only when interactivity is required (click handlers, state, effects).
3. **Every interactive component must handle three states:** loading, error, and empty.
4. **No business logic in components.** Delegate to API calls (`lib/api.ts`) or utility functions (`lib/utils.ts`).
5. **Use `STATUS_CONFIG` from `lib/constants.ts`** for all status rendering. Never hardcode status labels, colors, or icons in a component.
6. **Use `theme` from `lib/theme.ts`** for all layout and brand styling. Never hardcode brand colors in a component.

### Change-Friendliness Rules

These rules ensure that common changes require editing exactly one file:

| Change requested | File to edit | Files that should NOT change |
|-----------------|-------------|------------------------------|
| Change a status color or label | `lib/constants.ts` | No components change |
| Change brand color or sidebar style | `lib/theme.ts` | No components change |
| Add a new API endpoint | `lib/api.ts` + `lib/types.ts` | Existing components unchanged |
| Add a new page | `app/{route}/page.tsx` | No other pages change |
| Add a new reusable component | `components/{domain}/Name.tsx` | No existing components change |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamdez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
