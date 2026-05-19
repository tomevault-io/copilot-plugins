## ratemyplace

> > **Doc map:** [`MASTER.md`](MASTER.md) is the canonical product spec (mission, ethics, rating instrument, privacy, schema). [`ARCHITECTURE.md`](ARCHITECTURE.md) is the technical architecture reference. [`brand.md`](brand.md) is the voice and visual brand bible. [`CLAUDE_CONTEXT.md`](CLAUDE_CONTEXT.md) is the orientation doc for Claude Code agents. This file is coding conventions only.

# RateMyPlace Boston - Coding Conventions

> **Doc map:** [`MASTER.md`](MASTER.md) is the canonical product spec (mission, ethics, rating instrument, privacy, schema). [`ARCHITECTURE.md`](ARCHITECTURE.md) is the technical architecture reference. [`brand.md`](brand.md) is the voice and visual brand bible. [`CLAUDE_CONTEXT.md`](CLAUDE_CONTEXT.md) is the orientation doc for Claude Code agents. This file is coding conventions only.

## Quick Reference

| Item | Convention |
|------|------------|
| Framework | Astro 5.x SSR + React islands |
| Database | Cloudflare D1 (SQLite) |
| Auth | Lucia v3 with D1 adapter |
| Styling | Tailwind CSS 4.x |
| Types | TypeScript strict mode |

## File Patterns

### Pages (Astro)
- **Public pages**: `src/pages/*.astro` - SSR, no client JS unless needed
- **Dynamic routes**: `src/pages/[slug].astro` - Use `Astro.params.slug`
- **Admin pages**: `src/pages/admin/*.astro` - Always check `locals.user?.isAdmin`

### API Routes
- **Location**: `src/pages/api/**/*.ts`
- **Auth check pattern**:
```typescript
if (!context.locals.user) {
  return new Response(JSON.stringify({ error: 'Authentication required' }), {
    status: 401,
    headers: { 'Content-Type': 'application/json' }
  });
}
```
- **Admin check pattern**:
```typescript
if (!context.locals.user?.isAdmin) {
  return new Response(JSON.stringify({ error: 'Admin access required' }), {
    status: 403,
    headers: { 'Content-Type': 'application/json' }
  });
}
```

### Components
- **Astro components**: Static/SSR rendering, use for layouts and data display
- **React components**: Interactive islands only (forms, maps, dynamic UI)
- **React directive**: Always use `client:load` for immediate interactivity

```astro
<!-- Astro component with React island -->
<ReviewForm client:load buildingId={building.id} />
```

### Library Files (`src/lib/`)
- **Single responsibility**: One concern per file
- **Export types**: Always export interfaces for consumers
- **Critical files**:
  - `scoring.ts` - All scoring logic (weights, calculations)
  - `surveyItems.ts` - Survey questions and help text
  - `validation.ts` - Input validation
  - `audit.ts` - Admin action logging

## Database Patterns

### Getting DB Connection
```typescript
import { getDB } from '../../lib/db';

const db = getDB((context.locals as any).runtime);
```

### Query Patterns
```typescript
// Single row
const user = await db.prepare('SELECT * FROM users WHERE id = ?')
  .bind(userId)
  .first<User>();

// Multiple rows
const { results } = await db.prepare('SELECT * FROM reviews WHERE building_id = ?')
  .bind(buildingId)
  .all<Review>();

// Insert with generated ID
import { generateIdFromEntropySize } from 'lucia';
const id = generateIdFromEntropySize(10);
await db.prepare('INSERT INTO reviews (id, ...) VALUES (?, ...)')
  .bind(id, ...)
  .run();
```

### Timestamps
- Use `unixepoch()` for SQLite timestamps (not `datetime('now')`)
- Column type: `INTEGER DEFAULT (unixepoch())`

## Scoring System (Critical)

### Modifying Weights
1. Edit `ITEM_WEIGHTS` in `src/lib/scoring.ts`
2. Document justification with academic citation
3. Update `src/pages/methodology.astro`

### Adding Survey Items
1. Add column in new migration (`migrations/XXXX_name.sql`)
2. Add to `src/lib/surveyItems.ts` with help text
3. Add to domain array in `src/lib/scoring.ts` (UNIT_FIELDS, BUILDING_FIELDS, or LANDLORD_FIELDS)
4. Set weight in `ITEM_WEIGHTS`
5. Update `ReviewForm.tsx` and `ReviewCard.astro`

### Weight Guidelines
| Weight | Use For |
|--------|---------|
| 1.5x | Major health hazards (pests, mold) |
| 1.3x | Safety hazards (structural, climate) |
| 1.2x | Health-adjacent (plumbing, security) |
| 1.0x | Standard quality factors |

## Error Handling

### API Responses
```typescript
// Success
return new Response(JSON.stringify({ data: result }), {
  status: 200,
  headers: { 'Content-Type': 'application/json' }
});

// Client error
return new Response(JSON.stringify({ error: 'Validation failed', details: errors }), {
  status: 400,
  headers: { 'Content-Type': 'application/json' }
});

// Server error
return new Response(JSON.stringify({ error: 'Internal server error' }), {
  status: 500,
  headers: { 'Content-Type': 'application/json' }
});
```

### Audit Logging (Admin Actions)
```typescript
import { createAuditLog } from '../../lib/audit';

// Best-effort logging - failures don't break the action
await createAuditLog(db, {
  adminUserId: context.locals.user.id,
  actionType: 'review_approved',
  entityType: 'review',
  entityId: reviewId,
  oldValue: { status: 'pending' },
  newValue: { status: 'approved' }
});
```

## Styling

### Score Colors

**Single source of truth: [`src/lib/scoring-colors.ts`](src/lib/scoring-colors.ts).** Always import from there. Do not roll your own thresholds or labels.

Canonical four-band system (mirrors `brand.md` §4.2):

| Band | Range | Label | Fill | Text | Hex |
|------|-------|-------|------|------|-----|
| Good | 4.0–5.0 | `Good` | `bg-emerald-600` | `text-emerald-700` | `#059669` |
| Mixed | 3.0–3.9 | `Mixed` | `bg-amber-500` | `text-amber-700` | `#F59E0B` |
| Concerning | 2.0–2.9 | `Concerning` | `bg-amber-700` | `text-red-700` | `#A16207` |
| Poor | 1.0–1.9 | `Poor` | `bg-red-700` | `text-red-700` | `#B91C1C` |

Available helpers:
- `getScoreColor(score)` → `{ bg, text, label }` for filled badges/pills
- `getScoreTextColor(score)` → Tailwind class for colored score numbers
- `getScoreBgTint(score)` → soft-tinted background class for score detail tiles
- `getScoreHex(score)` / `SCORE_HEX` → hex strings for non-Tailwind contexts (Google Maps markers, OG images, PDF exports)

If you're adding a new surface that displays a score, reach for one of these. Never hardcode `text-teal-600` or `bg-orange-500` for score-band display.

### Brand Colors
- **Primary**: `text-teal-600` / `bg-teal-600`
- **Stars**: `text-amber-400`
- **Danger**: `text-red-600`

## Testing

### Run Tests
```bash
npm test              # All tests
npm test -- scoring   # Filter by name
```

### Test Location
- Unit tests: `src/lib/__tests__/*.test.ts`
- E2E tests: `e2e/*.spec.ts`

## Pre-Deploy QA Process

**Run this before every deploy. No exceptions.**

Before pushing any change to production, walk through every user-facing flow affected by the change and check all five categories below. If the change touches data display, check ALL pages where that data appears, not just the page you modified.

### 1. Display & UI Check
- Does any content overflow its container?
- Are internal variable names, database field names, or snake_case values visible to users?
- Do all counts, labels, and text display correctly (no "undefined", "null", "NaN", or empty strings where data should be)?
- Does the layout hold on mobile (375px), tablet (768px), and desktop (1280px)?

### 2. Data Consistency Check
- Does the same data (review counts, averages, scores, property details) show correctly across EVERY view where it appears (search results, property detail, admin panel, user profile)?
- Are calculated values (averages, aggregate scores) mathematically correct? Spot-check at least 3 properties.
- After adding/editing/deleting a review, do all views update to reflect the change?

### 3. Empty & Edge State Check
- What happens when search returns no results? Is the message helpful?
- What happens with an empty search query?
- What happens when a property has zero reviews?
- What happens when a user submits a form with missing or invalid data?
- What happens with extremely long text inputs?

### 4. Security Audit
- Are any API keys, database field names, or internal IDs exposed in the frontend HTML/JS?
- Can a non-authenticated user access authenticated routes by navigating directly to the URL?
- Can a non-admin user access admin routes?
- Are all database queries parameterized (no string interpolation with user input)?
- Do API error responses avoid leaking server details, file paths, or stack traces?
- Can the database be queried directly from the browser console or by manipulating frontend requests?

### 5. Search & Filter Logic
- Does search return the expected results for common queries?
- Does filtering narrow results correctly?
- Are results sorted as expected?
- Does pagination work (if applicable)?
- Is the result count accurate and consistent with what's displayed?

### When to Run a Full QA Pass
- Before every deploy to production
- After any change to database queries, scoring logic, or search
- After any change to components that appear on multiple pages
- After adding a new API endpoint

### QA Slash Command
Use `/qa` to run this checklist. Prompt: "Walk through every user-facing page and flow in this app. For each, check for display bugs, data consistency across views, empty/edge states, security vulnerabilities, and search/filter logic. Report everything you find, organized by category."

## Migrations

### Create Migration
```bash
# Create new migration file
touch migrations/XXXX_description.sql

# Apply locally
npx wrangler d1 migrations apply ratemyplace-db --local

# Apply to production
npx wrangler d1 migrations apply ratemyplace-db --remote
```

### Migration Naming
- Format: `XXXX_description.sql` (e.g., `0015_add_feature.sql`)
- Next number: Check existing migrations and increment

## Security Checklist

When adding new endpoints:
- [ ] Auth check if needed (`context.locals.user`)
- [ ] Admin check if admin-only (`context.locals.user?.isAdmin`)
- [ ] Input validation before processing
- [ ] Parameterized queries (never string interpolation)
- [ ] Rate limiting for public endpoints
- [ ] Audit logging for admin actions

### CSRF Protection

CSRF protection comes from three layers — no token implementation required (audited 2026-04-28):

- **SameSite=Lax** on session cookies (`src/middleware.ts`) and the OAuth state cookie (`src/pages/api/auth/google.ts`) — cross-site POSTs do not carry the session cookie, so authenticated endpoints are inherently protected
- **Cloudflare Turnstile** on every unauthenticated public POST form (signup, forgot-password, contact, disputes, bug-reports)
- **Astro `security.checkOrigin`** (default `true` for SSR) — rejects cross-origin form-content-type POSTs

**Important caveat:** `checkOrigin` does NOT cover `application/json` POSTs. `/api/disputes` accepts JSON, so its CSRF protection is Turnstile + rate limit + Phase 17 content-type guard, NOT checkOrigin. When adding a new JSON-accepting endpoint, ensure these three controls are wired.

Full audit: `.planning/audits/csrf-2026-04.md`. Re-audit triggers: Astro major version upgrade, Lucia replacement, new OAuth provider, or new non-form content-type endpoint.

## Common Mistakes to Avoid

1. **Don't use `datetime('now')`** - Use `unixepoch()` for timestamps
2. **Don't skip auth checks** - Every API route needs explicit auth handling
3. **Don't modify scoring without documentation** - Update methodology page
4. **Don't use `any` types** - Define interfaces in `types.ts`
5. **Don't put business logic in components** - Use `lib/` files
6. **Don't deploy without running `/qa`** - The QA process exists for a reason
7. **Don't assume one page is the only place data appears** - Always check all views that show the same data

## Git Workflow

### Commit Prefixes
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation
- `chore:` - Maintenance
- `refactor:` - Code restructuring

### Branch Strategy
- `main` - Production (auto-deploys to Cloudflare)
- Feature branches for development

---
*See `CLAUDE_CONTEXT.md` for full project context and architecture.*

---
> Source: [mereditharmcgee/ratemyplace](https://github.com/mereditharmcgee/ratemyplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
