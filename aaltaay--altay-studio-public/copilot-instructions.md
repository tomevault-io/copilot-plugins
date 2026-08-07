## altay-studio-public

> Altay Studio is an automated SaaS platform (the "Control Plane") that rapidly provisions, deploys, and manages isolated business websites. It acts as a website factory by creating an isolated schema in a master Supabase project, cloning a tailored GitHub repository (template), injecting environment variables (including VITE_DB_SCHEMA), and deploying to Vercel for every new client.

# Altay Studio AI Context & Governance

## Project Overview
Altay Studio is an automated SaaS platform (the "Control Plane") that rapidly provisions, deploys, and manages isolated business websites. It acts as a website factory by creating an isolated schema in a master Supabase project, cloning a tailored GitHub repository (template), injecting environment variables (including VITE_DB_SCHEMA), and deploying to Vercel for every new client.

---

## ⚠️ Critical: The Provisioning Flow (MUST READ)

Every time a client submits the "Launch My Site" form, the `provision-client` edge function executes these steps **in order**. Getting the order wrong has caused bugs multiple times — do not change it without reading this.

```
User submits form
      │
      ▼
1. AUTHENTICATE — verify the user's JWT via supabase.auth.getUser(token)
      │
      ▼
2. DB RECORD (INIT) — insert into public.businesses with owner_id, slug, schema_name,
                      provisioning_status = 'in_progress', provisioning_step = 'init'
      │
      ▼
3. DB SCHEMA — create a new Postgres schema (e.g. schema_ahmiclinic) in the
               master Supabase project and run the template's schema.sql into it
               (Update DB record: provisioning_step = 'db_schema')
      │
      ▼
4. GITHUB REPO — call POST /repos/{org}/{template-repo}/generate to clone the
                 template into a brand-new repo named {slug}-site
                 (e.g. aaltaay/template-barber → aaltaay/ahmiclinic-site)
                 ⚠️ Wait ~3 seconds after creation before the next step.
                 (Update DB record: github_repo, provisioning_step = 'github_repo')
      │
      ▼
4b. INJECT CONFIG — GET then PUT tenant.config.json into the new repo via
                    GitHub Contents API. Overwrites the template default with
                    tenant-specific colors, fonts, feature flags, and pages.
                    Non-fatal if it fails (env vars are the primary source).
      │
      ▼
5. VERCEL PROJECT — call POST /v10/projects?teamId={VERCEL_TEAM_ID} to create
                    a new Vercel project named {slug}, linking it to the NEW repo.
                    Inject all VITE_ env vars at this step.
                 (Update DB record: vercel_project_id, provisioning_step = 'vercel_deploy')
      │
      ▼
6. CLOUDFLARE DNS — DNS is handled automatically via a global wildcard CNAME 
                (*.altaystudio.com → cname.vercel-dns.com) on Cloudflare. 
                No per-client API calls are required for DNS.
                 (Update DB record: provisioning_step = 'done')
      │
      ▼
7. UPDATE DB — write vercel_deployment_url back onto the business record, 
               mark provisioning_status = "completed", provisioning_step = "done"
```

### Rollback Strategy
- If **any step fails** → Catch error, update DB record with `provisioning_status = 'failed'` and `provisioning_error = err.message`. This allows the Admin Dashboard to see the failure.
- External resources (like a partially created GitHub repo or Vercel project) should either be rolled back manually via the Admin Dashboard or via a robust retry mechanism. We currently keep the DB record as an error log.

### Template → Repo Name Convention
| Template repo      | Business type | New client repo name   |
|--------------------|---------------|------------------------|
| template-barber    | barber        | `{slug}-site`          |
| template-clinic    | clinic        | `{slug}-site`          |

Template is resolved from the `GITHUB_TEMPLATE_BARBER` / `GITHUB_TEMPLATE_CLINIC` secrets (NOT hardcoded). The `GITHUB_ORG` secret controls the GitHub org (default: `aaltaay`).

## The Admin Control Plane & Diagnostics

The platform includes an Admin Dashboard (`/admin`) for monitoring and debugging provisioning flows. It interfaces with two main Edge Functions:
1. `provision-client`: Handles the initial creation.
2. `admin-action`: A secure proxy function that takes an `action` and `business_id` to perform live diagnostics or cleanup.
   - **`diagnose`**: Connects to the database to check if the schema exists, pings GitHub to verify the repo exists, and hits Vercel API to check the project.
   - **`cleanup`**: Drops the schema (`DROP SCHEMA CASCADE`), deletes the GitHub repo, and deletes the Vercel project, allowing the admin to start over cleanly.

---

## Key Architectural Decisions & Gotchas
*   **Archival Architecture:** Adopted a "Safe Archive" philosophy. When an admin deletes a business record, the system relies on archiving resources instead of fully destroying them to prevent accidental data loss. The Edge Function renames the Postgres schema and the GitHub repository by prefixing them with `archived_` and a timestamp. Vercel projects are permanently deleted to free up domains, and the local business record is removed to clear the dashboard.
1. **The Template Contract (CRITICAL)**: Templates are separate repositories that must adhere to a strict contract. They must include:
   - `altay.config.json` (metadata, build commands, required env vars)
   - `supabase/schema.sql` (full database schema and RLS)
   - `.env.example`
   - `vercel.json` (for SPA routing rewrites)
   - `src/lib/supabase.ts` (dynamic Supabase client)
   - **MANDATORY FEATURES FOR EVERY WEBSITE (See blocks/core/CORE_FEATURES.md for details):**
     - **An Admin Route:** A secure `/admin` dashboard. Authentication must be simple: username `admin`, password `admin`.
     - **Footer Branding:** A footer that explicitly states "Designed by Altay Studio".
     - **SEO Ready:** Full-blown SEO capabilities, including `robots.txt`, sitemap generation, meta tags, and crawler optimization (`llms.txt`).
     - **Lead Generator:** A built-in lead generation mechanism or form to capture prospect information.
2. **Private Repositories by Default (CRITICAL)**: Every new GitHub repository generated by the platform MUST be private. Never generate a public repository for a client project.
3. **Dynamic Environment Variables**: Templates must *never* hardcode URLs, API keys, colors, or business names. All dynamic configuration must be injected via `NEXT_PUBLIC_` (Phase 3) and `VITE_` (legacy) environment variables at deploy time by the platform.
9. **GitHub Packages & NPM_TOKEN (CRITICAL)**: The `@altaystudio/core` package is published as `@aaltaay/altaystudio-core` to GitHub Packages. Templates use an npm alias (`"@altaystudio/core": "npm:@aaltaay/altaystudio-core@^1.0.0"`) so imports remain `@altaystudio/core`. Every Vercel project MUST have `NPM_TOKEN` (set to the GitHub token) as an encrypted env var, and the template must include a `.npmrc` file that references `${NPM_TOKEN}`.
10. **Vercel Framework Detection**: Provisioning engine MUST set `framework: "nextjs"` (not `"vite"`) when creating Vercel projects. Vercel uses this to set the correct output directory and build pipeline.
11. **Dual Env Var Injection**: The provisioning engine injects BOTH `VITE_*` (legacy) and `NEXT_PUBLIC_*` (Phase 3) env vars. This ensures backward compatibility with any remaining Vite-based templates while supporting the Next.js App Router architecture.
12. **No package-lock.json in Templates**: Templates MUST NOT include `package-lock.json` in the repo. Lockfiles with local `file:` references will break Vercel builds. npm generates a fresh lockfile during `npm install`.
4. **Template Registration**: New templates must be registered in `src/constants.ts` under the `TEMPLATE_MAP` before they can be provisioned.
5. **Vercel API Limits**: Because the platform automates Vercel API calls to create projects and set domains, be extremely mindful of rate limits during bulk testing or rapid provisioning.
6. **`VERCEL_TEAM_ID` is required**: All Vercel API calls must include `?teamId={VERCEL_TEAM_ID}` or the API returns 400. This is set as a Supabase secret.
7. **`owner_name` is NOT NULL**: The `businesses` table has a NOT NULL constraint on `owner_name`. The edge function must always include it in the insert.
8. **Platform Secrets & Environment Variables**: The control plane relies heavily on external API tokens which are documented in `.env.example`. These must be configured in the Supabase Edge Functions environment (`npx supabase secrets set`) or local `.env` depending on the execution context. Critical tokens include:
   - `GITHUB_TOKEN`, `GITHUB_ORG`, `GITHUB_TEMPLATE_BARBER`, `GITHUB_TEMPLATE_CLINIC`
   - `VERCEL_API_TOKEN`, `VERCEL_TEAM_ID`, `DYNADOT_API_KEY`, `RESEND_API_KEY`
   - `SUPABASE_DB_URL`, `ALTAY_DB_URL`, `SUPABASE_PUBLISHABLE_KEYS`, `SUPABASE_SECRET_KEYS`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_JWKS`, `SB_REGION`, `SB_EXECUTION_ID`, `DENO_DEPLOYMENT_ID`
   - `ANTHROPIC_API_KEY` (Claude 3.5 Sonnet — Agent OS ReAct loop)
   - `OPENAI_API_KEY` (text-embedding-3-small — Agent OS memory embeddings)

## The Agentic OS (Phase 4)

Every tenant website is equipped with an autonomous AI agent (the "Agent OS") that operates **behind the scenes**. It is NOT a chatbot — customers never see it. It wakes up on events (form submissions, missed calls, CRONs) and takes real-world actions (emails, SMS, CRM updates, content publishing).

### Architecture
```
Event (lead form, missed call, CRON, review)
      │
      ▼
Supabase Database Webhook → agent-director Edge Function
      │
      ├── 1. RESOLVE TENANT — query public.businesses for slug, config
      ├── 2. RECALL — vector search schema_{slug}.agent_memories
      ├── 3. REASON — Claude 3.5 Sonnet ReAct loop (pure REST, no SDK)
      ├── 4. ACT — execute tools (send_email, update_lead, store_memory)
      └── 5. RECORD — log full trace to schema_{slug}.agent_tasks
```

### Key Files
- `blocks/core/agent_os/schema.sql` — Per-tenant `agent_memories` (pgvector) + `agent_tasks` tables
- `blocks/core/agent_os/engine.ts` — Shared TypeScript types and tool definitions
- `blocks/core/agent_os/instructions.md` — System prompt templates per business type
- `supabase/functions/agent-director/index.ts` — The Edge Function (ReAct loop brain)
- `supabase/migrations/20260510000000_enable_pgvector.sql` — Global pgvector extension

### Critical Rules
1. **Memory isolation**: Each tenant's memories live in `schema_{slug}.agent_memories`. Memories NEVER leak between tenants.
2. **Pure REST**: The agent calls Claude via raw `fetch()` to `api.anthropic.com`. No LangChain, no Vercel AI SDK. Deno-native, zero dependencies.
3. **Audit trail**: Every agent action is logged in `schema_{slug}.agent_tasks` with the full LLM trace for debugging.
4. **Tool-based actions**: The agent MUST use tools (send_email, update_lead_status, store_memory) — it never "just responds with text".

## Headless Block Architecture (New Standard)

> **Full documentation**: See `blocks/README.md` in this repo root for the complete architectural reference.

### Core Concept
We have migrated away from bloated monolithic templates to an AI-Native **Headless Block Architecture**. Instead of copying a massive codebase and toggling features via `tenant_config.json`, the AI agent dynamically composes the site by injecting standardized "Blocks".

A Block contains exactly:
1. `schema.sql` (DB plumbing and RLS)
2. `engine.ts` (Headless React hook for state and API calls)
3. `instructions.md` (AI prompt to generate the bespoke Tailwind CSS Skin)

This completely decoupled approach drastically reduces AI token usage and ensures pristine, bespoke UI code for every tenant.

### Legacy Modular Template Architecture (Phase 2)
*(Kept for historical reference)*
Every business website runs from the SAME template codebase. The difference between tenants is a `tenant_config` JSON blob — not different code. Features are toggled via config, not by adding/removing files.

### tenant_config JSON (stored in `businesses.tenant_config`, injected as `VITE_TENANT_CONFIG`)
```json
{
  "features": {
    "booking_calendar": { "enabled": true, "required": true },
    "gallery": { "enabled": true },
    "staff_profiles": { "enabled": false }
  },
  "pages": ["home", "booking", "gallery", "contact"],
  "theme": { "primary_color": "#2D3A2E", "font_heading": "Playfair Display" }
}
```

### Key Files in Templates
- `altay.config.json` — declares supported features, defaults, required env vars
- `src/lib/features.ts` — reads `VITE_TENANT_CONFIG`, provides `isFeatureEnabled()`, `getActivePages()`, `getTheme()`
- `src/components/FeatureGate.tsx` — `<FeatureGate feature="gallery">` only renders children if feature is enabled
- `src/modules/{feature}/` — self-contained module directories (component + hook + exports)

### Critical Rules
1. **Module isolation**: Modules can NEVER import from or reference other modules. Only core config, shared UI, and Supabase client.
2. **Backward compatibility**: When `VITE_TENANT_CONFIG` is absent (legacy sites), all features default to ON.
3. **Required features**: Features marked `required: true` in `altay.config.json` cannot be disabled. The signup form locks them on.
4. **Config source of truth**: `businesses.tenant_config` in the database is the source of truth. `VITE_TENANT_CONFIG` env var is a copy injected at deploy time.

### Feature Registry (Control Plane)
`src/constants.ts` contains `FEATURE_REGISTRY` — a map of business_type → available features. The signup form reads this to show toggle switches.

### Platform Evolution
Phase 1 (completed): Config layer on top of template cloning.
Phase 2 (completed): Modular `src/modules/` layout in templates (Tailwind CSS v3, Shadcn UI, dynamic CSS variables, strict module isolation).
Phase 3 (completed): Extracted `@altaystudio/core` npm package + extremely lightweight tenant repos with `/custom` directory override mechanisms + automated Renovate Bot updates + `altay-eject` escape hatch.

### The "Bespoke-to-Template" Pipeline (Custom Sites & Scaling)
To handle completely standalone custom landing pages without breaking the control plane, we use the **Bespoke-to-Template Pipeline**:
1. **The Bespoke Phase (Do things that don't scale):** The Admin Dashboard supports a `bespoke` business type. This clones a blank shell repository (`template-bespoke`) that satisfies the strict Template Contract (`altay.config.json`, `supabase/schema.sql`, etc.) so the DB, GitHub, and Vercel are correctly wired. The developer then manually clones this shell, drops in the 100% custom React/Tailwind code, and pushes to Vercel. This allows testing highly custom designs to find product-market fit.
2. **The Template Phase (Scale to infinity):** Once a custom design (e.g., a specific bakery site) is proven to generate revenue, the developer extracts it into a generalized template (e.g., `template-bakery`), replacing hardcoded text/colors with variables from `VITE_TENANT_CONFIG`. This new template is added to `FEATURE_REGISTRY`, allowing the Admin Dashboard to instantly provision 10,000 instances of the proven design.

---

## Known Bugs Fixed

The full internal history of production incidents (provisioning bugs, PostgREST schema exposure, Tailwind build pipeline issues, etc.) has been condensed into [`docs/PLATFORM_GOTCHAS.md`](./docs/PLATFORM_GOTCHAS.md) — the ten most instructive root-cause/fix pairs, kept for anyone curious how this platform evolved.

## Continuous Updating
Whenever you learn something new about this project, encounter a bug and fix it, or make an architectural decision, **YOU MUST UPDATE THIS FILE** so future sessions have the necessary context.
*Rule Check*: Any API or any secret created for edge functions MUST be documented in `.env.example` and this file.
- **2026-05-10 (CRITICAL)**: Adding `vector(1536)` columns (pgvector) to tables in tenant schemas exposed to PostgREST caused the PostgREST schema cache introspection to hang indefinitely, killing ALL API access across the entire platform. Root cause: PostgREST's schema cache rebuild query becomes extremely expensive when introspecting `vector` type columns + HNSW indexes across multiple schemas. **RULE: NEVER add pgvector `vector()` columns to tables in schemas listed in `pgrst.db_schemas`.** Fixed by: (1) dropping `embedding` column and HNSW indexes from all tenant `agent_memories` tables, (2) killing PostgREST connections via `pg_terminate_backend()`. The permanent solution is to use direct SQL (postgres driver) for all vector operations — never expose them through the Supabase JS client / PostgREST.
  - **ALSO**: The `pgrst.db_schemas` setting was RESET during the crash recovery. It must be re-set to include tenant schemas before their sites work via the Supabase JS client.

## Workflow Rules
1. Before starting any task on this project, read this file to refresh your memory.
2. After completing a task, use the standard `git add .`, `git commit`, and `git push` protocol.
3. **Medical Client Provisioning:** Before onboarding any healthcare or medical spa client, you MUST review the `healthcare-compliance` Knowledge Item (KI) in your system to ensure all HIPAA, GDPR, and other regulatory requirements (e.g., BAAs, encryption, access logs) are fulfilled.
4. **Preserve Core Skills (Build Wrappers):** Do not modify or discard core skills/utility tools, as they are used across projects. Instead, build wrappers around them for specific use cases. The core stays untouched, while the wrapper is modified to fit the current needs.
5. **The Block Extraction Protocol (CRITICAL):** Whenever you are asked to build a new feature or component for a specific client site (e.g., a calendar, an ordering system, a custom hook), you MUST first check if it exists in the `blocks/` directory. If you build it from scratch or heavily modify it, you MUST extract the generic, reusable version of that feature back into the `blocks/` directory (including its `schema.sql`, `engine.ts`, and `instructions.md`) and update the Master Manifest in `blocks/README.md`. Never build the same feature twice.

## Karpathy Behavioral Guidelines
This project strictly enforces the Andrej Karpathy LLM principles to prevent common AI coding mistakes.
1. **Think Before Coding**: State assumptions, present tradeoffs, stop if confused.
2. **Simplicity First**: Write minimum viable code. No speculative abstractions.
3. **Surgical Changes**: Touch only requested lines. Leave unrelated code untouched.
4. **Goal-Driven Execution**: Define verifiable success criteria and loop until verified.

Before working on this project, ensure you adhere to these rules.

## Web Verification & Browser Testing
- **Web Verification**: At the end of every task involving web deployments or changes, agents MUST open a headless browser (using `agent-browser` or Playwright) and test the actual live subdomain URL (not localhost) to ensure it loads successfully and functions correctly before declaring the task complete.
- **Agent Browser CLI**: Use `npx agent-browser@latest` for fast, lightweight interaction.

---
> Source: [aaltaay/altay-studio-public](https://github.com/aaltaay/altay-studio-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
