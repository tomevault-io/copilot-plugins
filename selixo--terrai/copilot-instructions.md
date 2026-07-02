## terrai

> You are an agent working on **Terrai** — a SaaS that generates PDF reports on building plots in Poland. Before writing code, read this file in its entirety.

# Terrai — Agent Rules

You are an agent working on **Terrai** — a SaaS that generates PDF reports on building plots in Poland. Before writing code, read this file in its entirety.

--

## Stack and Versions

| Technology   | Version | Notes                             |
| ------------ | ------- | --------------------------------- | ------- |
| Next.js      | 16.x    | App Router, **not** Pages Router  |
| React        | 19.x    | Server Components by default      |
| TypeScript   | 5.x     | `strict: true`, zero `any`        |
| Tailwind CSS | 4.x     | New syntax - `tailwind.config.js` | missing |
| Supabase     | latest  | Auth + Postgres + RLS             |
| Claude API   | latest  | `claude-sonnet-4-5` for reports   |

---

## Folder structure

```
app/
(auth)/ # /login, /register, /reset-password
(dashboard)/ # /dashboard, /reports/[id]
(marketing)/ # /, /price list, /demo
API/
reports/ # POST /api/reports - starts the pipeline
webhooks/ # POST /api/webhooks/stripe
components/
ui/ # Atomic: Button, Input, Badge, Card...
report/ # ReportSection, SectionStatus, ScoreBar...
map/ # LeafletMap, ParcelPicker...
lib/
ai/ # Claude API prompts and calls
api/ # Integrations: uldk.ts, isok.ts, geoportal.ts...
db/ # Supabase queries (typed)
pdf/ # React-PDF templates
validators/ # Zod schemas — the ONLY validation source
types/
report.ts # ReportSection, ReportStatus, ReportScore
parcel.ts # ParcelData, ParcelIdentifier
api.ts # Request/Response types for Route Handlers
```

---

## Coding Rules

### TypeScript

- Zero `any`. If you don't know the type, use `unknown` and narrow it down.
- Zero `// @ts-ignore` without a comment explaining why.
- Export types from `types/` — do not define inline in components.
- Prefer `type` over `interface` (unless extending).

### Components

- One file = one component.
- **Named export**, never `export default`.
- Server Component default — `'use client'` only when needed (event handlers, hooks, browser APIs).
- Type props inline as `type Props = { ... }` over the component.

### Route Handlers (API)

- Each endpoint **validates input with Zod** — no validation = review error.
- Return `NextResponse.json({ error: '...' }, { status: 4xx })` on errors.
- API secrets (Anthropic, Supabase service role) — server-side only, never in the client.
- Support `Promise.allSettled` for parallel external API calls — partial failure = graceful degradation.

### Database

- Queries only via `lib/db/` — never inline in components or Route Handlers.
- Row Level Security enabled on every table — the agent must take this into account when writing policies.
- Types generated from Supabase (`database.types.ts`) — don't write table types manually.

### Environment Variables

- `NEXT_PUBLIC_` — only what **must** be present in the client (Supabase URL, anon key, app URL).
- Everything else — no prefix, available only in Server Components and Route Handlers.
- The `.env.example` pattern is the source of truth — always update `.env.example` when adding a new variable.

---

## Report generation pipeline

```
POST /api/reports
→ Zod validation (parcelIdentifier)
→ ULDK API → full cadastral identifier
→ Promise.allSettled([
fetchEGiB(), // area, soil class, ownership
fetchISOK(), // flood zones
fetchGeoportal(), // MPZP, Natura 2000, power lines
fetchPIG(), // geology, groundwater
fetchOSM(), // POI, roads, services
])
→ buildReportPayload(results) // normalizes data, marks gaps
→ callClaudeAPI(payload) // 3 grouped calls
→ generatePDF(sections) // React-PDF
→ uploadToR2(pdf) // Cloudflare R2
→ saveReport(db) // Supabase
→ sendEmail(user) // Resend
```

**Key rule:** Any external API can fail. Use `Promise.allSettled` — never `Promise.all` for external data. Mark missing data as `data_missing`, don't terminate the entire pipeline.

--

## AI Contract — Report Sections

### Types (types/report.ts)

```typescript
export type SectionStatus = 'green' | 'yellow' | 'red'

export type ReportSection = {
status: SectionStatus
summary: string // 1-2 sentences — visible in preview
narration: string // full paragraph to PDF
red_flags: string[] // empty if there are no contraindications
data_missing: string[] // which data failed to download
}

export type ReportScore = {
overall: number // 0-10, one decimal place
legal: number
infrastructure: number
physical: number
environment: number
location: num
```

---
> Source: [Selixo/terrai](https://github.com/Selixo/terrai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
