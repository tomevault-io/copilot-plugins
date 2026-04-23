## dropsites

> > **This is the single authoritative entry point.** Read this file before writing any code.

# CLAUDE.md — DropSites

> **This is the single authoritative entry point.** Read this file before writing any code.
> For detailed docs, see [`docs/README.md`](docs/README.md).

## What is DropSites

DropSites is a self-hosted static site publishing platform. Drop a file or folder in, get a shareable URL out. It serves HTML, JS apps, React builds, PDFs, and any browser-renderable static content. Built for consultants, PMs, developers, and AI power users who need a fast, reliable way to publish and share without technical overhead.

## Tech Stack (Locked)

| Technology | Role | Key Constraint |
|---|---|---|
| Next.js 15 (App Router) | Full-stack framework | App Router only, no Pages Router |
| shadcn/ui (Radix + Tailwind) | UI component library | Must use — never rebuild covered surfaces |
| Tailwind CSS v4 | Utility-first styling | Required by shadcn |
| Geist (Vercel) | Typography | Weights 400 and 500 only — no other fonts |
| Supabase (PostgreSQL) | Database + RLS + Auth | RLS enforces all workspace permissions at DB level |
| Supabase Auth | Authentication | Magic link, Google, GitHub OAuth — zero custom auth Phase 1 |
| Cloudflare R2 | Deployment file storage | Zero egress — S3-compatible API only |
| Cloudflare CDN | Edge serving + TLS + cache | Cache invalidation within 30s on overwrite |
| Resend | Transactional email | Configurable via env var |
| Twilio | SMS notifications | Pay-per-message, security/time-sensitive only |
| Sentry | Error monitoring | Free tier sufficient at launch |
| Vitest | Unit + integration tests | Jest-compatible, native ESM |
| Playwright | E2E tests + accessibility (axe-core) | Multi-browser, milestone gate verification |

## Repository Structure

```
dropsites/
├── app/                        # Next.js App Router pages and API routes
├── components/
│   ├── ui/                     # shadcn/ui (generated — do not hand-edit)
│   └── marketing/              # Marketing page components
├── lib/                        # Business logic, utilities, storage abstraction
├── styles/                     # tokens.css, primitives.css (design tokens)
├── docs/                       # All documentation (see docs/README.md for index)
│   ├── prd/                    # Product requirements
│   ├── design/                 # Design system, component specs
│   ├── architecture/           # System architecture (reserved)
│   ├── implementation/         # Plan, progress, task cards
│   ├── feature-contracts/      # Feature contract schemas + .yaml contracts
│   ├── milestones/             # Milestone gate definitions (reserved)
│   └── reference/              # HTML prototypes, design artifacts
├── scripts/                    # Validation and enforcement scripts
├── supabase/                   # Schema, migrations (created during M1.1)
├── tests/                      # Unit, integration, E2E tests
├── public/                     # Static assets
├── CLAUDE.md                   # This file
└── package.json
```

## Key Architectural Rules

1. **Never hardcode limits** — all resource checks call `getProfile(workspaceId)`. No numeric limits anywhere in code.
2. **Never modify uploaded source files** — auto-nav, CSP, robots headers, lazy-loading, password protection injected at serve time via middleware.
3. **All storage via S3-compatible abstraction only** — no direct R2 SDK calls in business logic. Backend selected via `STORAGE_BACKEND` env var.
4. **RLS enforces all workspace permissions** — never rely on application-layer checks alone. DB rejects unauthorized queries.
5. **No credentials in code** — all secrets via environment variables or mounted secrets volume.
6. **Use shadcn components** — never custom implementations for covered surfaces. See [`docs/design/components.md`](docs/design/components.md).
7. **Geist font only** — weights 400 (regular) and 500 (medium). No other fonts.
8. **Design tokens only** — no hardcoded colour, spacing, or sizing values in components. All tokens in `styles/tokens.css`. See [`docs/design/design-system.md`](docs/design/design-system.md).
9. **Every icon-only button must have a Tooltip** — no exceptions.
10. **Mobile-first** — fully functional at 375px minimum viewport. All actions reachable without horizontal scroll.
11. **Lucide React icons** — 16px in tables, 20px in toolbars, stroke-width 1.5px, no filled icons.
12. **Single accent colour** — orange-500 (`#f97316`) for primary buttons, active nav, focus rings only. Never decorative.

## Documentation

All docs live in `docs/` with an index at [`docs/README.md`](docs/README.md).

| What | Where |
|------|-------|
| Product requirements | [`docs/prd/PRD.md`](docs/prd/PRD.md) |
| Design system (tokens, colours, typography, theming) | [`docs/design/design-system.md`](docs/design/design-system.md) |
| Component specs (40+ components) | [`docs/design/components.md`](docs/design/components.md) |
| Implementation plan (358 tasks, 61 milestones) | [`docs/implementation/PLAN.md`](docs/implementation/PLAN.md) |
| Progress tracker | [`docs/implementation/PROGRESS.md`](docs/implementation/PROGRESS.md) |
| Session task cards (S01–S44) | [`docs/implementation/TASK_CARDS.md`](docs/implementation/TASK_CARDS.md) |
| Feature contract format | [`docs/feature-contracts/SCHEMA.md`](docs/feature-contracts/SCHEMA.md) |

## Database

Key tables (full schema created during M1.1):

| Table | Purpose |
|---|---|
| `users` | Accounts — email, limit_profile, trial dates, notification prefs |
| `workspaces` | Org containers — name, namespace, owner, limit_profile, Stripe sub |
| `workspace_members` | Membership — user + workspace + role (owner/publisher/viewer) |
| `deployments` | Published sites — slug, workspace, entry_path, storage, password, expiry |
| `analytics_events` | View events — deployment, timestamp, referrer domain, UA class (no PII) |
| `bandwidth_daily` | Daily aggregate bytes served + request count per deployment |
| `audit_log` | Append-only log — publish, delete, role change, key operations |

## Current Milestone

**Phase 3 complete. All sessions S74–S78 done.** DropSites v1.0 GA — MCP server published, beta infrastructure live, smoke tests passing.

_Update this line at the start of each new milestone session._

## How to Work

1. **Read this file** and the current milestone doc before writing any code.
2. **State which milestone and FRs** you are working on at session start.
3. **Read [`docs/implementation/PROGRESS.md`](docs/implementation/PROGRESS.md)** to understand what's already built.
4. **Write implementation first, then tests.** Never tests without implementation.
5. **Run milestone gate tests** before marking any milestone complete.
6. **Update the Current Milestone** line above when advancing.
7. **Run `npx tsc --noEmit`** before every commit — zero type errors.
8. **Run the design system verification checklist** (in [`docs/design/design-system.md`](docs/design/design-system.md) Section 8) before marking any UI milestone complete.
9. **Every PR must pass:** unit tests, integration tests, E2E tests, axe accessibility scan, `npm audit` zero critical.
10. **After any UI change,** verify at 375px and 1280px viewport widths.
11. **After marking any session complete in PROGRESS.md,** commit and push to remote — do not leave completed work unpushed.

## Enforcement

Automated checks run on every commit:

| Check | Script | Blocks Commit? |
|-------|--------|----------------|
| TypeScript compilation | `npx tsc --noEmit` | Yes |
| Design token compliance | `scripts/validate-design-tokens.ts` | Yes |
| Doc link integrity | `scripts/validate-docs.ts` | Yes |
| Lint | `npm run lint` | Yes |

## Testing

**Stack:** Vitest (unit/integration) · Playwright + axe-core (E2E + accessibility) · Lighthouse CI (performance >= 90) · k6 (load).

200+ named test cases defined in PRD Section 11.

**Gate rule:** No milestone is complete until all its gate tests pass. Gates are binary — no exceptions, no partial credit.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/estegonza2002) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
