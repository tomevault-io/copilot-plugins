## interact

> <!-- Last verified: 2026-03-16 -->

<!-- Last verified: 2026-03-16 -->
# GEMINI.md — Gemini CLI / Gemini Agent Context

> Context file for Google Gemini CLI (`gemini` CLI tool) and Gemini-powered coding agents.
> Keep this file updated when architecture, conventions, or commands change.

---

## Project Summary

**Interact** is an enterprise-grade employee engagement and gamification platform (v0.0.0, active development).
It is a React 18 + Vite 6 single-page application backed by Base44 SDK serverless functions running on Deno.
The platform delivers points, badges, leaderboards, AI-powered activity recommendations, and team analytics
for organizations improving employee retention and cohesion.

**Repository:** `Krosebrook/interact`
**Canonical docs:** `docs/README.md` → all documentation navigation
**Architecture:** See [ARCHITECTURE.md](./docs/ARCHITECTURE.md) and [docs/adr/](./docs/adr/)

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| UI Framework | React | 18.2.0 |
| Build Tool | Vite | 6.1.0 |
| Language | JavaScript (JSX) — TypeScript migration Q2 2026 | — |
| Styling | TailwindCSS + Radix UI primitives | 3.4.17 |
| State (server) | TanStack Query | 5.84.1 |
| State (UI) | React Context API | — |
| Forms | React Hook Form + Zod | 7.54.2 / 3.24.2 |
| Animations | Framer Motion | 11.16.4 |
| Backend | Base44 SDK (Deno serverless functions) | 0.8.20 (frontend) / 0.8.6 (Deno) |
| Auth | Base44 Auth (JWT) | — |
| Testing | Vitest + React Testing Library | 4.0.17 |
| CI/CD | GitHub Actions → Vercel | — |
| Node.js | 20 (LTS) | >=20 required |

---

## Key Directories

```
src/
  App.jsx               — Router setup, provider composition
  Layout.jsx            — App shell (nav, sidebar, auth guard)
  main.jsx              — Vite entry point
  pages.config.js       — Auto-generated static page map (127 pages)
  pages/                — 127 route-level page components
  components/           — 64 feature component sub-directories
    ui/                 — Radix UI / shadcn primitives
    common/             — Shared utility components
    gamification/       — Points, badges, leaderboards
    analytics/          — Charts and dashboards
    ai/                 — AI-powered components
    docs/               — Auto-generated in-app docs (EXCLUDED from lint)
  api/
    base44Client.js     — Base44 SDK client (uses VITE_BASE44_APP_ID, VITE_BASE44_BACKEND_URL)
  lib/
    AuthContext.jsx     — useAuth hook and auth context
    query-client.js     — TanStack Query client config (staleTime: 5 min)
    utils.js            — cn() helper for Tailwind class merging
    imageUtils.js       — Cloudinary URL optimization
    app-params.js       — Env param resolution
  hooks/                — Custom React hooks (prefix: use*)
  utils/                — Pure utility functions
  test/                 — Test setup, fixtures, and unit tests
functions/              — ~200 Deno/TypeScript serverless functions
docs/                   — Structured documentation (architecture, ADRs, guides)
ADR/                    — Root-level ADR copies (canonical: docs/adr/)
scripts/                — CLI and tooling scripts
.github/workflows/      — ci.yml, docs-authority.yml, safe-merge-checks.yml
```

---

## Environment Variables

### Frontend (Vite `VITE_` prefix — bundled into browser build)

| Variable | Required | Description |
|---|---|---|
| `VITE_BASE44_APP_ID` | ✅ | Base44 application identifier |
| `VITE_BASE44_BACKEND_URL` | ✅ | Base44 backend URL (e.g. `https://api.base44.app`) |
| `VITE_COMPANY_ID` | Optional | Multi-tenant company identifier |
| `VITE_ENABLE_ANALYTICS` | Optional | Feature flag — analytics (`true`/`false`) |
| `VITE_ENABLE_PWA` | Optional | Feature flag — PWA mode (`true`/`false`) |
| `VITE_GOOGLE_ANALYTICS_ID` | Optional | Google Analytics GA4 ID |
| `VITE_STRIPE_PUBLISHABLE_KEY` | Optional | Stripe public key (payments) |
| `VITE_SENTRY_DSN` | Optional | Sentry error tracking DSN |

**Rule:** Never put secrets in `VITE_` variables — they are exposed to the browser.

### Backend (Deno functions — set in Base44 dashboard)

| Variable | Required | Description |
|---|---|---|
| `OPENAI_API_KEY` | ✅ (AI features) | OpenAI GPT-4 |
| `ANTHROPIC_API_KEY` | ✅ (AI features) | Anthropic Claude 3 |
| `GOOGLE_API_KEY` | Optional | Gemini Pro |
| `SLACK_BOT_TOKEN` | Optional | Slack integration |
| `STRIPE_SECRET_KEY` | Optional | Stripe payments |
| `TWILIO_ACCOUNT_SID` | Optional | SMS via Twilio |
| `CLOUDINARY_CLOUD_NAME` | Optional | Media uploads |
| `SENDGRID_API_KEY` | Optional | Email notifications |

---

## Build & Run Commands

```bash
# Install dependencies (CI-safe, exact versions)
npm ci

# Development server
npm run dev                  # http://localhost:5173

# Production build
npm run build

# Preview production build
npm run preview

# Lint (must pass before commit)
npm run lint                 # check
npm run lint:fix             # auto-fix

# Tests
npm run test:run             # single pass (CI mode)
npm test                     # watch mode
npm run test:coverage        # with coverage report

# Type check
npm run typecheck

# Generate PRD
npm run prd:generate
```

---

## Code Conventions

### Component Structure

```javascript
// Order: imports → constants → hooks (TOP LEVEL ONLY) → handlers → JSX
import { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import { toast } from 'sonner';
import { cn } from '@/lib/utils';

export default function ComponentName({ prop1, className }) {
  const [state, setState] = useState(null);

  useEffect(() => { /* side effects */ }, [dependencies]);

  const handleAction = async () => {
    try {
      toast.success('Done');
    } catch (error) {
      toast.error('Something went wrong');
    }
  };

  return <div className={cn('p-4', className)}>{/* JSX */}</div>;
}
```

### Naming

| Artifact | Convention | Example |
|---|---|---|
| React component files | PascalCase | `ActivityCard.jsx` |
| Custom hooks | `use` prefix + camelCase | `useActivityData.js` |
| Utility functions | camelCase | `imageUtils.js` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Backend functions | camelCase | `awardBadgeForActivity.ts` |
| Boolean vars | `is`/`has`/`can` prefix | `isLoading`, `hasError` |
| Event handlers | `handle` prefix | `handleSubmit` |

### Data Fetching (TanStack Query)

```javascript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { base44 } from '@/api/base44Client';

const { data, isLoading } = useQuery({
  queryKey: ['activities'],
  queryFn: () => base44.entities.Activity.list(),
  staleTime: 5 * 60 * 1000,  // 5 minutes — default
});
```

Query key pattern: `['resource']` → `['resource', id]` → `['resource', { filters }]`

### Backend Function Pattern (Deno)

```typescript
import { createClientFromRequest } from 'npm:@base44/sdk@0.8.6';

Deno.serve(async (req) => {
  try {
    const base44 = createClientFromRequest(req);
    const user = await base44.auth.me();
    if (!user) return Response.json({ error: 'Unauthorized' }, { status: 401 });
    // validate inputs, business logic...
    return Response.json({ success: true, data: result });
  } catch (error) {
    console.error('Function error:', error);
    return Response.json({ error: 'Internal server error' }, { status: 500 });
  }
});
```

### Error Handling

- Wrap all async operations in `try/catch`
- Show generic `toast.error('Something went wrong')` to users
- Log detailed errors to console (never expose stack traces in responses)
- Use `<ErrorBoundary>` from `src/components/common/ErrorBoundary.jsx` for React render errors

### Import Order

1. React core (`react`, `react-dom`)
2. Third-party libraries (`@tanstack/react-query`, `framer-motion`)
3. UI components (`@/components/ui/button`)
4. Feature components (`@/components/activities/ActivityCard`)
5. Utilities (`@/lib/utils`, `@/lib/imageUtils`)
6. Icons (`lucide-react`)

---

## Architecture Constraints

- **Hooks at top level only** — never inside conditions, loops, or after early returns (ESLint enforced)
- **No moment.js** — use `date-fns` instead (`import { format } from 'date-fns'`)
- **TailwindCSS only** — no inline styles, no separate CSS files
- **`cn()` for conditional classes** — import from `@/lib/utils`
- **Server state in TanStack Query** — never in useState for API data
- **UI/app state in React Context** — for auth, theme, notifications
- **Zod validation** on all forms (frontend) and all function inputs (backend)
- **DOMPurify** before any `dangerouslySetInnerHTML`
- **`async/await`** throughout — avoid mixing with `.then()/.catch()`
- **No hardcoded secrets** — all via environment variables

---

## Security Constraints

- Every Deno backend function MUST call `base44.auth.me()` and check for null before any operation
- Strip sensitive fields (`password`, `api_key`, tokens) before returning data from functions
- Validate all user inputs in backend functions (Zod or manual validation)
- Never log sensitive data (passwords, tokens, PII) to console
- Role-based access: check `user.role` in functions for admin-only operations
- Run `npm audit` before releases; address high/critical issues

---

## Common Tasks

### Add a new page
1. Create `src/pages/NewPage.jsx` using existing page as template
2. Base44 auto-registers the page in `src/pages.config.js` — do **not** edit that file manually
3. Add route to `src/App.jsx`

### Add a new backend function
1. Create `functions/functionName.ts` following the Deno pattern above
2. Add authentication check as first step
3. Validate all inputs before processing

### Add a new form
1. Define Zod schema: `const schema = z.object({...})`
2. Use `useForm({ resolver: zodResolver(schema) })`
3. Use `<Form>`, `<FormField>`, `<FormItem>`, `<FormMessage>` from `@/components/ui/form`

### Add a test
1. Create `ComponentName.test.jsx` in same directory as component
2. Wrap with `QueryClientProvider` if component uses TanStack Query
3. Follow AAA pattern: Arrange → Act → Assert

---

## Known Constraints & Gotchas

1. **`src/pages.config.js` is auto-generated** — static imports, not `React.lazy`. Do not manually add lazy loading here.
2. **ESLint flat config** ignores `src/components/docs/**` and `src/components/lib/architecture/**` — these are auto-generated doc files.
3. **`VITE_BASE44_API_URL` is NOT required** — only `VITE_BASE44_APP_ID` and `VITE_BASE44_BACKEND_URL` are needed for the frontend client.
4. **`npm test` runs in watch mode** — use `npm run test:run` in CI/scripts.
5. **`moment` is listed as a dependency** but should not be used — use `date-fns` for all new date formatting.
6. **ADRs exist in two locations**: `ADR/` (root, 5 ADRs) and `docs/adr/` (canonical, 9 ADRs). Always use `docs/adr/`.
7. **Sentry is not yet integrated** — error tracking is TODO for production.
8. **31 tests are currently skipped** in `functions/tests/` — these require a live Base44 environment.

---

## Files / Directories — Do NOT Modify Without Approval

- `src/components/docs/**` — auto-generated in-app documentation
- `src/components/lib/architecture/**` — auto-generated architecture files
- `src/pages.config.js` — auto-generated page map
- `package-lock.json` — modify only via `npm install` or `npm ci`
- `.github/workflows/**` — CI pipeline definitions
- `vercel.json` — deployment configuration

---

## Related Agent Files

- [CLAUDE.md](./CLAUDE.md) — Claude Code / Anthropic agent context
- [AGENTS.md](./AGENTS.md) — All 21 specialized Copilot sub-agents
- [.github/copilot-instructions.md](./.github/copilot-instructions.md) — GitHub Copilot inline rules
- [MEMORY.md](./MEMORY.md) — Persistent project history and current state

---

**Last Updated:** 2026-03-16
**Source of truth for stack:** `package.json`, `src/api/base44Client.js`, `src/lib/app-params.js`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Krosebrook) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
