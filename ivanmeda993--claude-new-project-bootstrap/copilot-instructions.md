## claude-new-project-bootstrap

> Bootstrap a NEW project from scratch using Ivan's preferred stack and conventions, then write production-ready first feature code. Triggered when Ivan describes what he wants to build ("napravi mi novi projekat za X", "treba mi nova app", "novi monorepo", "scaffold a Next.js app", "new project for ...", "start a new SaaS", "imam ideju za app"). For fuzzy ideas, enters Phase 0 (discovery) — uses Tavily MCP for competitor research and Context7 MCP for library docs. For web apps, lets user pick Next.js or TanStack Start. For Next.js with BE logic, asks how to handle backend (Server Actions, embedded Elysia, tRPC, or separate service). Phase 3 confirms the stack and offers an optional add-ons menu (T3 env, Resend, Stripe, Upstash, Sentry, next-themes, UploadThing, next-safe-action, pino, etc.) — user opts in. Phase 3.5 resolves LATEST stable versions via Context7 / `pnpm view`. Phase 6 (MANDATORY for any first feature code) reads `references/code-patterns.md` (extracted from real Inertia code) and invokes complementary skills (`react-hook-form`, `tanstack-query`, `shadcn-ui`, `next-best-practices`) — every form uses RHF `<Controller>` + custom `<Field>/<FieldGroup>/<FieldLabel>/<FieldError>` (NOT shadcn `<Form>`), every mutation uses inline `useMutation` (not custom hook) wrapping `unwrap(clientApi.x.y())`, query keys use two-tier factory (entity + feature-local), server prefetch via `<PrefetchBoundary queries={[...Options()]}>`, money is `string` in/out (Decimal in DB), no raw `useState` forms, no inline `fetch()`, no inline query keys, no local FormField helpers, no monolith feature files (≤150 lines target, split form/preview/view). When a feature uses a library or primitive NOT covered in code-patterns.md (Stripe, react-pdf, drag-and-drop, OAuth flows, file upload, etc.), the skill MUST research first via Context7 + Tavily MCP, document the new pattern in the project's .claude/rules/, THEN write code — never improvise on unknown libraries. Picks a stack archetype (single-app web, full-stack monorepo, design-system monorepo, mobile-first, API-only), then scaffolds folder structure, config files, AGENTS.md, .claude/, package.json scripts, and tooling — all matching Ivan's existing conventions extracted from inertia, inka, magic-digs-monorepo. Do NOT invoke for retrofitting existing projects (use ai-project-init for that).


# New Project Bootstrap

Use when Ivan wants to **start a brand-new project**. The skill picks a tech stack and folder layout based on what he's building, using his actual habits across past projects (not generic best-practices).

## When NOT to use

- Existing project being prepared for Claude → use `ai-project-init`
- Adding a feature to an existing project → use module/feature-specific skills
- Just exploring ideas, no commitment to scaffold → answer the question, do not run this skill

## Tools to use (and when)

Default to MCP-aware research instead of guessing. Check what's available in the current session:

| Need                                                                                                       | Tool                                                 | When                                                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Library / framework / SDK docs (current API, version-specific)                                             | `context7` MCP (`resolve-library-id` + `query-docs`) | ALWAYS prefer over WebFetch when the question is about a library, framework, or SDK. Even for well-known ones like React, Next.js, Prisma — training data is stale. |
| Competitive / market / inspiration research, "what apps exist for X", "how do others solve Y", recent news | `tavily` MCP (`tavily_search`, `tavily_research`)    | When user wants idea exploration, competitor analysis, or general web search. Prefer over WebFetch for unknown URLs.                                                |
| Specific known URL the user provided                                                                       | `WebFetch`                                           | Only when user gives a direct URL to read                                                                                                                           |
| Generic web search with no MCP available                                                                   | `WebSearch`                                          | Last resort if neither MCP is available                                                                                                                             |

**Rule**: before any web research, check what MCPs are exposed in the session. If `tavily_*` or `context7` tools appear, use them. Don't fall back to WebFetch/WebSearch if a better-targeted MCP exists.

## Phase 0 — Discovery & brainstorm (when the idea is fuzzy)

If the user's project description is vague ("nešto za fitnes", "neki AI tool", "marketing site za moj startup"), DO NOT jump to Phase 1. First, work with the user to sharpen the idea.

### When to enter Phase 0

Enter Phase 0 if any of the following:

- The pitch is one short sentence with no clear product shape
- User explicitly asks for help thinking through the idea ("pomozi mi da razradim", "nisam siguran šta tačno")
- The target user / core feature / monetization model is unclear
- The user wants to research what already exists before deciding

### What Phase 0 covers

Have a real conversation, not an interrogation. Aim for 5-10 minutes of back-and-forth covering:

1. **Core value** — what does the user actually solve, and for whom? Force a one-sentence answer.
2. **Differentiator** — what makes it not just "another X"? (If there's no answer, that's a flag — say so.)
3. **MVP scope** — what's the smallest thing that could prove the idea? List 3-5 features max.
4. **Out of scope (v1)** — what is explicitly NOT in the first version. This is as important as what's in.
5. **Target users** — who, how many, what device, what context (mobile-first vs desktop-first)
6. **Data shape** — what entities exist, who owns them, what's the privacy model
7. **Monetization (if any)** — free, freemium, B2B, ads, no money yet

### Tools during Phase 0

- **Tavily** — competitor scan ("what apps exist for habit tracking with social features"), market signals, recent funding/news on the space. Use `tavily_research` for deep dives, `tavily_search` for quick lookups.
- **Context7** — only if user mentions a specific library/SDK they want to use ("hoću da koristim X SDK"); confirm it's still alive and check current API.
- **AskUserQuestion** — for branching choices (mobile-first vs web-first, free vs paid).

Use Tavily/Context7 PROACTIVELY when relevant — don't make the user ask. If they say "ne znam šta već postoji", run a Tavily scan and summarize.

### Output of Phase 0

A short brief, written into the conversation (not a file unless user asks):

```
## <Project name>

**One-liner**: <single sentence>
**MVP scope**: <3-5 bullets>
**Out of scope (v1)**: <bullets>
**Target user**: <who>
**Platform**: <web / mobile / both>
**Auth**: <yes/no, what kind>
**Data**: <main entities>
**Monetization**: <model or "none yet">
**Inspiration / competitors**: <2-3 names if researched>
```

Then ask "Da li ovo dobro opisuje? Ako da, idem na predlog tech stack-a." Wait for confirmation before Phase 1.

## Phase 1 — Pick the stack (max 3 questions)

Once the idea is clear (either obvious from the start, or refined in Phase 0), ask **at most 3** targeted questions via `AskUserQuestion`. Skip any question whose answer is already obvious from the brief.

**Q1 — What is it?** (always ask if not obvious)

- Web app (SaaS, dashboard, marketing site)
- Full-stack with separate API (web + api + maybe mobile)
- Component / design-system library
- Mobile app (Expo)
- API-only service / SDK

**Q2 — What's the deployment target?** (only if relevant)

- Render (default for self-managed simplicity)
- Vercel (Next.js shortcut, but Ivan prefers Render)
- Self-hosted Docker / Kubernetes (Inka pattern, with Helm)
- TBD / decide later

**Q3 — Auth needed?** (only if user app, not library)

- Better Auth (default for new projects, especially with mobile)
- NextAuth (Next.js single-app, OIDC providers like Keycloak)
- None / public / API key

### Archetype-specific extra questions

Some archetypes need additional clarity that doesn't fit in Q1-Q3. Ask these AFTER the first three, only when the archetype is selected.

#### Web archetypes (`single-next`, `fullstack-monorepo` web app, `design-system-monorepo` docs) — Framework choice

For any web app, ask:

> **Koji React framework?**
>
> - **Next.js** (default) — App Router + RSC + Server Actions + Route Handlers. Najveći ekosistem, najviše integracija (Vercel, Render Node runtime), shadcn ide direktno. Bira se kada: koristiš Server Actions / RSC, hostuješ na Vercel/Render, treba ti SSG ili ISR.
> - **TanStack Start** — Vite-based React framework od TanStack tima. Type-safe routing kroz TanStack Router, server functions umesto Server Actions, tighter integracija sa TanStack Query. Bira se kada: dolaziš iz Vite/SPA sveta, voliš TanStack ergonomics, ne treba ti RSC, hoćeš lakše hostovanje (Node/Bun, Cloudflare Workers).
>
> Za marketing sites + SSG → Next.js (zreliji ISR/SSG i CDN priče).
> Za dashboard-heavy apps koji žive iza login-a → oba dobra; biraj po preference.

Pravilo: NIKAD ne forsiraj jedan framework ako user nije rekao. Ako odluči TanStack Start, koristi `tanstack-start` varijantu archetype-a iz `references/archetypes.md`.

#### `single-next` — Backend strategy (MUST ask if BE logic is needed)

If the user app needs any backend logic (forms that hit DB, mutations, API endpoints, integrations), ask:

> **Kako želiš da handle-uješ backend logiku?**
>
> - **Server Actions + Route Handlers** — najsimpler, sve u Next.js. Server Actions za mutations from forms, Route Handlers (`app/api/*/route.ts`) za webhooks/REST. Default za većinu SaaS-a.
> - **Embedded Elysia (Inka pattern)** — Elysia mounted on `app/api/[[...slugs]]/route.ts`. Daje OpenAPI, Eden Treaty type-safety, plugins. Bira se kad: API treba da bude tipovan kao "real" API, postoji šansa za mobile app kasnije, ili user voli Elysia ergonomics.
> - **tRPC** — type-safe RPC između Next i klijenta. Bira se ako user eksplicitno traži tRPC ili ako tim već zna tRPC. NE default.
> - **Separatni API service** — ako BE bude ozbiljan, prebaci u `fullstack-monorepo` archetype umesto da forsiraš single-app. Reci useru da ovo nije više `single-next`.

Pravilo: NIKAD ne pretpostavi BE strategiju za `single-next`. Različite strategije znače dramatično različitu folder strukturu, scripts, i AGENTS.md sadržaj.

#### `fullstack-monorepo` — confirm Eden Treaty

Default za API ↔ web kontrakt je Eden Treaty. Pitaj samo ako user pomene tRPC ili gRPC eksplicitno; inače idi sa Eden Treaty.

#### `mobile-only` — confirm API source

Pitaj: **Da li već postoji API ili treba da scaffolduje i API?**

- "Postoji backend" → samo bearer-token client, configure base URL.
- "Treba i API" → predloži da prebacimo u `fullstack-monorepo` umesto čistog `mobile-only`.

Match Ivan's language. He writes Serbian — answer in Serbian, code/configs in English.

## Phase 2 — Pick the archetype

Use the answers to pick **one** archetype from `references/archetypes.md`. Read that file in full before composing.

| User intent                                            | Archetype                                                              |
| ------------------------------------------------------ | ---------------------------------------------------------------------- |
| "novi SaaS / dashboard / marketing site, jedna app"    | `single-next` (Next.js single-app)                                     |
| "treba mi web + api + mobile" / "polni stack monorepo" | `fullstack-monorepo` (Turborepo + apps/{web,api,mobile} + packages/\*) |
| "design system" / "component library" / "shared UI"    | `design-system-monorepo` (Turborepo + packages/ui + apps/docs)         |
| "samo mobile" / "Expo app"                             | `mobile-only` (Expo + Unistyles + TanStack Query)                      |
| "API only" / "samo backend" / "SDK"                    | `api-only` (Elysia/Bun single-app or library)                          |

If the user's intent doesn't match cleanly, say so and propose the closest archetype. Do NOT silently force-fit.

## Phase 3 — Confirm the proposed stack

Output a **brief** stack proposal in Serbian (matching Ivan's language) and ask for confirmation before writing files. Format:

```
Predlog za <archetype>:
- Frontend: <Next.js | TanStack Start> + React 19 + TypeScript strict
- UI: shadcn/ui + Radix + Tailwind v4 + lucide-react
- Forms: React Hook Form + Zod + @hookform/resolvers
- State: TanStack Query v5 + nuqs (URL state, web only)
- DB: Prisma + PostgreSQL
- Auth: <chosen>
- Tooling: pnpm + Turbo (if monorepo)
- Lint: ESLint flat config + Prettier
- Hooks: Husky + lint-staged + commitlint conventional

Da scaffolduje, ili dodajemo nešto?
```

### Optional add-ons menu (offer BEFORE scaffolding)

After the base stack proposal, offer the most useful add-ons and let the user pick. **Do not silently include them** — let the user opt-in. Format the question as a checklist:

> Hoćeš li nešto od ovoga?
>
> **Env validation**
>
> - `@t3-oss/env-nextjs` (Next.js) ili `@t3-oss/env-core` (Elysia/Bun) + Zod — type-safe env vars sa runtime check-om. Highly recommended.
>
> **Email**
>
> - `resend` + `@react-email/components` — transactional email kroz React komponente. Default ako treba slati mejl.
>
> **Payments**
>
> - `stripe` (Stripe SDK) ili `@lemonsqueezy/lemonsqueezy.js` — biraj jedan ako treba subscription/billing.
>
> **Rate limiting & Redis**
>
> - `@upstash/ratelimit` + `@upstash/redis` — serverless-friendly rate limiting.
>
> **Analytics**
>
> - `posthog-js` (full product analytics) ili `@vercel/analytics` (lightweight pageview).
>
> **Error tracking**
>
> - `@sentry/nextjs` (single-next) ili `@sentry/node` (api) — production error reporting.
>
> **Theming**
>
> - `next-themes` (Next.js dark/light mode) — već uključeno ako se UI zatraži, inače opt-in.
>
> **File uploads**
>
> - `uploadthing` + `@uploadthing/react` — najbrži put za image/file uploads. Alternativa: direktno na S3/R2.
>
> **OG images**
>
> - `@vercel/og` ili `satori` + `next/og` — dynamic OpenGraph slike.
>
> **PDF generation**
>
> - `@react-pdf/renderer` — PDF iz React komponenti, server-side.
>
> **Logging (API)**
>
> - `pino` + `pino-pretty` — structured logging za Elysia. Default ako se gradi production API.
>
> **Background jobs**
>
> - `bullmq` + Redis (single-next sa long-running jobs) ili `inngest` (serverless workflow). Pitaj samo ako user pomene "background jobs", "queue", "scheduled task".
>
> **Date utils**
>
> - `date-fns` (modular) ili `dayjs` (Inka pattern, smaller). `date-fns` je default; biraj `dayjs` ako user voli moment-style API.

Pravilo: NE forsiraj sve od ovoga. Pomeni samo kategorije relevantne za projekat (npr. ne pominji PDF za marketing site, ne pominji payments za internal dashboard). Ako user kaže "sve recommended", uključi: T3 env, Sentry, Resend, next-themes, pino (za API).

### Override semantics

User može da:

- **Skine** nešto iz default stack-a ("ne treba mi commitlint", "skini Prisma")
- **Doda** add-on iz menu-a ("dodaj T3 env i Resend")
- **Doda** nešto van menu-a ("dodaj recharts", "treba mi `pdf-lib`")
- **Promeni** default ("dayjs umesto date-fns", "MUI umesto shadcn" — push back jednom, pa poslušaj)

Wait for confirmation pre nego što kreneš na Phase 3.5.

## Phase 3.5 — Resolve latest versions (MANDATORY)

**Before** writing a single config file, resolve the latest stable version of every dependency you're about to pin. Training data is stale; pinning hardcoded versions ships outdated stacks on day one.

### Tool order

1. **Context7 MCP** (`resolve-library-id` + `query-docs`) — preferred. Pulls live latest version + current API.
2. **`pnpm view <package> version`** (Bash) — fallback when Context7 doesn't index the package, or for fast batch resolution.
3. **`npm view <package> version`** — last resort (slower than pnpm).

### What to resolve

For every archetype, resolve the latest stable for at least:

**Always**: `typescript`, `prettier`, `eslint`, `@typescript-eslint/parser`, `@typescript-eslint/eslint-plugin`, `husky`, `lint-staged`, `@commitlint/cli`, `@commitlint/config-conventional`

**If web/Next**: `next`, `react`, `react-dom`, `tailwindcss`, `@tailwindcss/postcss`, `eslint-config-next`

**If monorepo**: `turbo`

**If API**: `elysia`, `@elysiajs/openapi`, `@elysiajs/cors`, `bun-types`

**If mobile**: `expo`, `expo-router`, `react-native`, `react-native-unistyles`

**If state/forms**: `@tanstack/react-query`, `react-hook-form`, `@hookform/resolvers`, `zod`, `nuqs`

**If DB**: `prisma`, `@prisma/client`

**If auth**: `better-auth`, `next-auth` (whichever was picked)

**If UI**: `@radix-ui/react-*` (only the ones you'll actually scaffold), `lucide-react`, `class-variance-authority`, `clsx`, `tailwind-merge`

### How to do it efficiently

Batch the lookups in a single `pnpm view` call where possible. Example shell:

```bash
for pkg in next react react-dom typescript prettier eslint tailwindcss prisma @prisma/client zod react-hook-form @tanstack/react-query nuqs lucide-react; do
  echo "$pkg@$(pnpm view "$pkg" version 2>/dev/null)"
done
```

Then write the resolved versions into `package.json`. **Do NOT** use the version strings in `references/archetypes.md` — those are illustrative ("Next.js 16+") and will rot.

### Versioning style in `package.json`

- Use **caret ranges** (`^16.2.4`) — not exact pins, not wildcards
- For TypeScript and tools that break on minor: use **tilde** (`~5.7.2`) only if user explicitly asks for stricter
- Same caret rule for peer deps in shared packages

### What to do if a package is yanked / broken / pre-release

- If latest is a `0.x` or has `-alpha`/`-beta`/`-rc`, DROP back to the last stable (use `pnpm view <pkg> versions --json | tail` and pick the last non-prerelease).
- If a package returns 404 on `pnpm view`, flag it — it may have been renamed or moved orgs.

### Don't skip this phase

If the user is rushing ("samo skafolduj brzo"), skip the scaffold itself before skipping version resolution. Stale `package.json` versions are the #1 reason a fresh scaffold fails to install or boots with security advisories.

## Phase 4 — Scaffold

Read `references/folder-structures.md` for the chosen archetype, then write files in this order:

1. **Root config**: `package.json` (with scripts AND resolved-latest dep versions from Phase 3.5), `tsconfig.json`, `pnpm-workspace.yaml` (if monorepo), `turbo.json` (if monorepo), `.gitignore`, `.npmrc`, `.editorconfig`
2. **Lint/format**: `.prettierrc`, `.prettierignore`, `eslint.config.mjs` (or shared `packages/eslint-config/` for monorepo)
3. **Git hooks**: `.husky/pre-commit`, `.husky/commit-msg`, `.lintstagedrc.json`, `commitlint.config.mjs`
4. **Folder structure**: `src/` (single-app) or `apps/` + `packages/` (monorepo) — see `references/folder-structures.md`
5. **AGENTS.md + CLAUDE.md**: AGENTS.md is canonical (see `assets/agents-md/<archetype>.md`), CLAUDE.md is just `@AGENTS.md`
6. **`.claude/` setup**: copy `assets/claude-config/` (settings.json + minimal commands/rules)
7. **`.mcp.json`**: minimal config with context7
8. **`README.md`**: short overview, install + dev commands, link to AGENTS.md

Use `references/config-files.md` for the canonical content of every config file. Do NOT freestyle.

## Phase 5 — Post-scaffold

After writing files:

1. Print a short summary of what was created (file count by category, not full list)
2. Print the **next 3 commands** the user should run:
   ```
   cd <project>
   pnpm install
   pnpm dev
   ```
3. If the archetype includes Prisma, add: `pnpm db:generate && pnpm db:push`
4. If mobile, add: `pnpm --filter <app> prebuild`
5. Remind: "Pre prvog commita: `git init && git add . && git commit -m 'chore: initial scaffold'`"

Do NOT auto-run `git init`, `pnpm install`, or any side-effect command. Ivan picks when to run those.

## Phase 6 — First feature code (production-ready, MANDATORY)

If the user asks for ANY first feature (login form, signup, dashboard skeleton, first CRUD route) — either as part of the same conversation as the scaffold or right after — **MUST follow `references/code-patterns.md` to the letter**. No exceptions, no shortcuts, no "simple version".

### Rule

**Before writing any `.tsx` / `.ts` file inside `src/modules/**`, `apps/web/module/**`, `apps/mobile/src/module/**`, or `apps/api/src/routes/**`for the first time, read`references/code-patterns.md` IN FULL.** Do not skim. Do not assume you remember.

### Mandatory invocations during Phase 6

When the agent writes the first feature, it MUST proactively invoke these complementary skills (via the `Skill` tool) before generating code:

| Code involves                                       | Skills to invoke (mandatory)                |
| --------------------------------------------------- | ------------------------------------------- |
| Any form                                            | `react-hook-form` + `shadcn-ui`             |
| Any data fetch                                      | `tanstack-query`                            |
| Any Tailwind / shadcn UI                            | `tailwind-v4-shadcn` + `shadcn-ui`          |
| Any Next.js boundary (RSC, route handler, metadata) | `next-best-practices`                       |
| Any new feature module structure                    | `frontend-patterns`                         |
| Auth, payments, user input                          | `security-review`                           |
| Database migration                                  | `database-migrations` + `postgres-patterns` |
| API contract / REST                                 | `api-design` + `backend-patterns`           |

### Hard pre-flight checklist (cannot skip)

Before writing the first form/page/route, the agent MUST verify these files exist (create if missing):

- [ ] `src/lib/schemas/<resource>.ts` — Zod schema with inferred types
- [ ] `src/lib/query-keys.ts` — query-key factory (web)
- [ ] `src/lib/errors.ts` — custom error classes
- [ ] `src/lib/db.ts` — Prisma singleton (if Prisma)
- [ ] `src/lib/auth.ts` + `src/lib/auth-client.ts` — auth setup (if auth)
- [ ] `src/lib/env.ts` — T3 env validation (if T3 add-on)
- [ ] `src/lib/utils.ts` — `cn()` helper (if shadcn)
- [ ] shadcn primitives installed: `pnpm dlx shadcn@latest add input button label card select` (NOT `form` — Ivan uses Controller + custom Field)
- [ ] `components/ui/field.tsx` — `Field/FieldGroup/FieldLabel/FieldError` family copied from `references/code-patterns.md` § 1
- [ ] `<Toaster />` mounted in root layout
- [ ] `error.tsx` per route group (Next.js)
- [ ] `loading.tsx` per async route group (Next.js)

If ANY are missing, create them FIRST. Don't write the feature without them.

### Forms — the golden pattern (no exceptions)

Ivan's actual stack is **`<Controller>` from RHF + custom `<Field>/<FieldGroup>/<FieldLabel>/<FieldError>` from `@/components/ui/field`** (NOT shadcn's `<Form>` context). Read `references/code-patterns.md` § 1 in full for the exact `Field` family code and an end-to-end example. Quick summary:

```tsx
// ✅ CORRECT — Controller + Field/FieldGroup/FieldLabel/FieldError
<form onSubmit={form.handleSubmit((v) => mutation.mutate(v))} noValidate>
  <FieldGroup>
    <Controller
      name="name"
      control={form.control}
      render={({ field, fieldState }) => (
        <Field data-invalid={fieldState.invalid}>
          <FieldLabel htmlFor="goal-name" required>
            Name
          </FieldLabel>
          <Input {...field} id="goal-name" aria-invalid={fieldState.invalid} />
          {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
        </Field>
      )}
    />
    {/* repeat per field — Select, Combobox etc. all wrapped in Controller */}
  </FieldGroup>
  <Button type="submit" disabled={mutation.isPending}>
    {mutation.isPending ? "Creating..." : "Create"}
  </Button>
</form>
```

```tsx
// ❌ WRONG — raw register + local FormField helper component
<form onSubmit={form.handleSubmit(onSubmit)} noValidate>
  <FormField label="Account" error={errors.account?.message}>
    <Input {...form.register("account")} />
  </FormField>
</form>
```

The wrong pattern duplicates `<Field>`, breaks Select/Combobox/DatePicker controlled state, and skips the `data-invalid` + `aria-invalid` wiring. Always `<Controller>` + `<Field>`. The skill MUST install the `Field` family from `code-patterns.md` § 1 into `components/ui/field.tsx` BEFORE writing the first form.

### Mutations — useMutation always (no exceptions)

Even for "simple" sync transformations (e.g. encoding form values into a QR string), wrap in `useMutation` if there's any chance it might:

- Become async later
- Need loading state
- Need error toast
- Trigger cache invalidation

```tsx
// ✅ CORRECT
const encodeMutation = useMutation({
  mutationFn: async (values: UplatnicaInput) => encodeUplatnica(values),
  onSuccess: (qrString) => setQrPayload(qrString),
  onError: (e) => toast.error(e.message ?? "Neuspešno generisanje QR-a"),
});

form.handleSubmit((v) => encodeMutation.mutate(v));
```

```tsx
// ❌ WRONG — try/catch + setState, no loading, no retry, no centralized error
function onSubmit(values: UplatnicaInput) {
  try {
    const { qrString } = encodeUplatnica(values);
    setQrPayload(qrString);
  } catch (e) {
    toast.error(e instanceof Error ? e.message : "...");
  }
}
```

**The only exception**: pure derived state with no side-effects (e.g. `const filtered = useMemo(() => items.filter(...), [items])`). That's not a mutation.

### Pattern gap protocol — when `code-patterns.md` doesn't cover the case

If the user asks for a feature that uses a library, primitive, or interaction that **isn't covered** in `references/code-patterns.md` (e.g. Stripe Elements, react-pdf, drag-and-drop, OAuth flow with Better Auth social providers, file upload via UploadThing, infinite scroll, virtualized lists, real-time WebSocket, etc.):

**STOP. Do NOT improvise.** Improvising on an unknown library is the #1 source of "looks plausible but breaks at runtime" code. Instead, follow this protocol.

#### Step 1 — Detect the gap

Before writing the first line of code, the agent verifies:

- Does `code-patterns.md` mention this library / primitive / interaction by name? (`rg -i "<term>" references/code-patterns.md`)
- Is there a relevant complementary skill installed? (e.g. `claude-api` for Anthropic SDK, `figma` for Figma MCP, `tanstack-query` for queries)

If both are NO → gap detected. Move to Step 2.

#### Step 2 — Research first (mandatory)

Use tools in this order:

1. **Context7 MCP** — preferred for library/SDK/API docs:
   - `mcp__plugin_context7_context7__resolve-library-id` with the library name
   - `mcp__plugin_context7_context7__query-docs` with the resolved ID and a precise question (e.g. "Stripe Checkout server action with webhook signature verification")
   - Always specify version + framework context if the lib has variants (e.g. `@sentry/nextjs` not just "sentry")

2. **Tavily MCP** — for "how do people actually use this in production" + recent gotchas + version-specific quirks:
   - `mcp__claude_ai_Tavily__tavily_research` for deep dives ("Stripe webhook in Next.js App Router 2026 best practices")
   - `mcp__claude_ai_Tavily__tavily_search` for quick lookups
   - Always include the year and framework name to filter stale results

3. **WebFetch** — only when the user provides a direct URL (e.g. official docs link).

4. **WebSearch** — last resort if no MCP available.

5. **Existing complementary skill** — invoke via `Skill` tool BEFORE writing code if relevant (`claude-api`, `tanstack-query`, `react-hook-form`, `database-migrations`, `security-review`, etc.).

#### Step 3 — Synthesize a project-local pattern

Once research is complete, compose the pattern that fits Ivan's stack (Controller + Field, useMutation, query-keys factory, etc.). Use the canonical Inertia conventions as the skeleton — the new library plugs INTO that skeleton, not the other way around.

Example shape: a Stripe payment form is still RHF + Zod + `<Controller>` + `<Field>` for the customer details + a `useMutation` wrapping `unwrap(clientApi.payments.createSession.post(...))` — Stripe Elements is just the visual primitive inside one `<FormItem>`.

#### Step 4 — Document the new pattern (write to project)

Before writing the feature code, add a short pattern doc into the **project's** `.claude/rules/` (NOT into the global skill — that lives in `~/.claude/skills/new-project-bootstrap/`). File name: `.claude/rules/<topic>-pattern.md` (e.g. `stripe-checkout-pattern.md`).

Required content:

- One-line summary
- Library + version pinned
- Citations (Context7 doc ID, Tavily URLs)
- Canonical code snippet adapted to Ivan's stack
- Anti-patterns to avoid (gotchas from research)
- When to invoke (trigger keywords)

This makes the new pattern reusable for the next feature without re-researching.

#### Step 5 — Then write the feature code

Now (and only now) write the actual feature code, citing the new pattern doc. Run the standard anti-pattern greps after.

#### When to extend `code-patterns.md` (the global skill)

If the new pattern is **stack-universal** (would apply to ANY new project Ivan starts, not just this one), the agent should propose adding it to the global skill:

> "Treba li ovaj <X> pattern da dodam u globalni `code-patterns.md` da bude dostupan u svakom budućem projektu? (`git pull` u `~/.claude/skills/new-project-bootstrap/`)"

Skip this offer if the pattern is project-specific (e.g. domain-specific business logic, one-off integrations).

#### Anti-pattern: improvising on unknown libraries

Never write code for a library you haven't verified via Context7 or current docs in this session. Training data on libraries is months stale at minimum. Common landmines:

- Stripe SDK: API versions change behavior (e.g. `payment_intent` vs `checkout_session`)
- Better Auth: plugin config differs per version
- Resend: from address verification rules differ by domain status
- Plaid: token flow + webhook verification (covered separately by `inertia-plaid` skill if Plaid is in play)
- next-safe-action: v7 vs v8 API is incompatible

If the agent finds itself thinking "I think the API is something like..." — STOP and run Context7.

### Cross-check after writing

After writing the first feature code, run these greps and FIX what's flagged:

```bash
# No raw useState forms
rg "useState.*''" src/ | rg -i "email|password|name|address"

# No fetch in components
rg "await fetch\(" src/modules src/app

# No process.env outside env.ts
rg "process\.env\." src/ | rg -v "src/lib/env.ts"

# All forms have noValidate
rg "<form " src/ | rg -v "noValidate"

# All shadcn forms use FormField context
rg "form\.register\(" src/
```

If any grep returns matches in committed code, refactor before moving on.

## Conventions extracted from Ivan's projects

These are mechanical rules — apply by default unless the user opts out.

### Files & folders

- **Files**: kebab-case always (`query-keys.ts`, `dashboard-content.tsx`, `auth-client.ts`)
- **Folders**: kebab-case always (`module/dashboard`, `api-client`)
- **Components**: PascalCase exports inside kebab-case files (`export function DashboardContent()`)
- **Hooks**: `use-*.ts` filename, `useX` export
- **Workspace packages**: `@<org>/<name>` (`@acme/ui`, `@acme/schemas`, `@acme/db`) — pick org from project name
- **No barrel exports** (no `index.ts` re-exports unless package boundary)
- **Route groups** (Next.js): parens — `(dashboard)/`, `(auth)/`

### Module organization (feature-first)

```
src/modules/<feature>/        # OR apps/web/module/<feature>/
  components/                 # reusable pieces (server or client)
  views/                      # page-level composition (server components)
  lib/                        # feature-local helpers (query-keys, queries, parsers)
  hooks/                      # feature-local hooks
```

- Subfolders only when needed — don't pre-create empty folders
- Cross-module shared → `src/components/` and `src/hooks/`

### TypeScript baseline

- `strict: true`, `noUncheckedIndexedAccess: true`, `isolatedModules: true`, `moduleDetection: force`, target ES2022
- Single path alias: `@/* → ./src/*`
- `consistent-type-imports` enforced (inline `import type {}`)
- No `any` unless justified by a comment

### Lint plugins (always)

- `simple-import-sort`
- `unused-imports`
- `eslint-plugin-only-warn` (warnings, not errors — don't block builds on style)
- `@typescript-eslint`
- For Next.js: `eslint-config-next/core-web-vitals` + `/typescript`
- For monorepo: `eslint-plugin-turbo`

### Prettier defaults

- Inertia/Inka style: semis ON, single quotes, ES5 trailing comma, 2-space, LF
- Magic-digs style: no semis, double quotes, 80-col, `prettier-plugin-tailwindcss` with `tailwindFunctions: ["cn", "cva"]`
- **Default for new project**: pick Inertia style (semis + single quotes) unless user says otherwise — most consistent with his recent work

### Validation

- **Zod always**, never alternative validators
- Schemas live in `packages/schemas` (monorepo) or `src/lib/schemas/` (single-app)
- Infer types from schemas: `type Foo = z.infer<typeof fooSchema>`
- Never duplicate TS interface for what a Zod schema already defines

### Forms

- React Hook Form + `@hookform/resolvers/zod` (always paired)
- No native browser validation — `noValidate` on every `<form>`

### Data fetching

- TanStack Query v5 always (server state)
- nuqs for URL state (web only — never mobile)
- Server-side prefetch + `<HydrationBoundary>` pattern in Next.js App Router
- Query key factory pattern: never inline `queryKey: ['foo', id]`

### Money / financial fields

- **Decimal as string** in Zod, `Decimal @db.Decimal(12, 2)` in Prisma — never `number` / `Float`
- Plaid context exception: integer cents (see Inertia's plaid-integration.md)

### Error handling

- Custom error classes in `src/lib/errors.ts` (NotFoundError, UnauthorizedError, ForbiddenError, ValidationError)
- Consistent shape: `{ error: string, code: string }`
- 404 (not 403) for ownership failures — don't leak existence

### AGENTS.md doctrine

- AGENTS.md is canonical, hand-written, lives at root + per-app + per-package
- CLAUDE.md is a 1-line shim: `@AGENTS.md`
- README.md is for humans (install, dev, links to AGENTS.md), not for AI

### MANIFEST.md (when reuse risk is real)

- Add `MANIFEST.md` per package when 3+ helpers risk duplication
- Schema: `name`, `import`, `purpose`, `inputs`, `outputs`, `replaces`, `tags`
- Skip for single-app projects unless they grow large
- See `references/manifest-template.md`

### .claude/ baseline

Every project gets:

```
.claude/
  commands/        # slash commands (start empty, add as needed)
  rules/           # project-scoped rules (start with README.md only)
  skills/          # local skills (start empty)
  hooks/           # PostToolUse / PreToolUse shell scripts
  plans/           # planning docs (gitignored)
  settings.json    # allowlist + denylist for tools
  README.md        # explains the layout
```

And: `.claude/plans/` in `.gitignore`.

### Commit conventions (when commitlint included)

- Format: `<type>(<scope>): <subject>` lowercase imperative, no trailing period
- Types: feat, fix, docs, style, refactor, perf, test, chore, ci, revert
- Scopes: per-app + per-package + `deps`, `config`
- Header max 100 chars

### Git hooks (Husky 9)

- `pre-commit` → `pnpm lint-staged`
- `commit-msg` → `pnpm commitlint --edit $1` (only if commitlint included)
- lint-staged: ESLint + Prettier on `.{ts,tsx,js,jsx,mjs}`, Prettier on `.{json,md,yml,yaml,css,scss}`

### Deployment defaults

- **Render** (`render.yaml` with `buildFilter.paths` for selective deploy in monorepo)
- **Docker** support always (`Dockerfile` + `.dockerignore`)
- **Bitbucket Pipelines + Helm + K8s** is the Inka pattern — only if user asks for self-hosted

## Files in this skill

| File                               | Purpose                                                                                                                                                               |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SKILL.md`                         | This file (entry point)                                                                                                                                               |
| `references/archetypes.md`         | 5 archetypes with stack composition                                                                                                                                   |
| `references/folder-structures.md`  | Folder layout per archetype                                                                                                                                           |
| `references/config-files.md`       | Canonical content of every config file                                                                                                                                |
| `references/naming-conventions.md` | File/folder/symbol naming cheatsheet                                                                                                                                  |
| `references/manifest-template.md`  | MANIFEST.md schema + when to add                                                                                                                                      |
| `references/code-patterns.md`      | **MANDATORY**: production-ready code patterns (RHF + shadcn Form, useMutation, query keys, T3 env, errors, Better Auth, server actions). Read in FULL before Phase 6. |
| `assets/configs/`                  | Drop-in config files (.prettierrc, tsconfig base, etc.)                                                                                                               |
| `assets/agents-md/<archetype>.md`  | AGENTS.md template per archetype                                                                                                                                      |
| `assets/claude-config/`            | `.claude/` baseline (settings.json, README.md, etc.)                                                                                                                  |

## Anti-patterns (do NOT do)

- ❌ Hardcode tech-stack choices without confirming with user
- ❌ Use Yarn or npm — always pnpm
- ❌ Use NativeWind in mobile — Ivan uses Unistyles v3
- ❌ Use Pages Router in Next.js — App Router only
- ❌ Use `axios` — use Eden Treaty (monorepo) or `fetch` (single-app)
- ❌ Use Joi / Yup / class-validator — Zod only
- ❌ Use Redux / MobX — TanStack Query for server state, Zustand for client state if truly needed
- ❌ Use Material UI in new projects unless specifically asked (Inka uses MUI for legacy reasons)
- ❌ Use camelCase or snake_case for filenames — kebab-case only
- ❌ Skip AGENTS.md or use README.md as the AI doc
- ❌ Auto-run `git init`, `pnpm install`, or any command that changes user state without asking
- ❌ Default to TypeScript without `strict` and `noUncheckedIndexedAccess`

---
> Source: [ivanmeda993/claude-new-project-bootstrap](https://github.com/ivanmeda993/claude-new-project-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
