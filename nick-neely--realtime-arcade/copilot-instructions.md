## realtime-arcade

> name: realtime-arcade


# Purpose: Give the AI and developers a single source of truth for architecture, conventions, and workflows

project:
name: realtime-arcade
description: >
Multi-game realtime web app with a shared hub, using Next.js 15 (App Router), TypeScript, Tailwind, shadcn/ui,
Supabase Realtime (broadcast, presence, postgres_changes), and Drizzle for schema and migrations.
goals: - Keep game modules pluggable via a registry - Prefer optimistic UI with authoritative DB confirmation - Never paste SQL into Supabase editor, use Drizzle migrations

stack:
runtime: nodejs 20
framework: nextjs 15
language: typescript strict
styling: tailwindcss, shadcn-ui
data:
primary: supabase postgres
orm: drizzle-orm + postgres-js
realtime: supabase realtime (presence, broadcast, postgres_changes)
state:
server_state: tanstack react-query
client_state: zustand (or jotai for atomic local state)
auth: supabase auth (email link, socials optional)
hosting: vercel

directories:
root: - src/app/ # App Router routes, server actions - src/app/(public)/login/ # unauth pages - src/app/dashboard/ # hub - src/app/play/ # public rooms list - src/app/games/[slug]/[roomId]/ # game shell pages - src/components/ # UI and shared components - src/components/ui/ # shadcn generated - src/db/ # drizzle schema, client, seeds - src/db/schema.ts # domain schema - src/db/client.ts # pooled db client for server - src/db/run-migrations.ts # migrator entry - src/db/seed.ts # seed script - src/lib/ # framework glue - src/lib/supabase/ # supabase client and server helpers - src/lib/auth/ # helpers like requireUser - src/lib/games/registry.ts # pluggable game catalog - src/games/<slug>/Client.tsx # each game client entry - src/hooks/ # custom hooks - src/store/ # zustand or jotai stores - drizzle/ # generated SQL migrations - public/ # static assets - scripts/ # any one-off scripts
create_on_demand: - src/games/<slug>/server/ # optional game server actions - src/games/<slug>/db.ts # optional game-specific queries - src/games/<slug>/schema.ts # optional game-specific tables

env:
required: - NEXT*PUBLIC_SUPABASE_URL - NEXT_PUBLIC_SUPABASE_ANON_KEY - SUPABASE_SERVICE_ROLE_KEY # server only - DRIZZLE_DATABASE_URL # non pooled for migrations - SUPABASE_DB_POOL_URL # pooled for runtime via pgbouncer - NEXT_PUBLIC_SITE_URL
rules: - Never expose SUPABASE_SERVICE_ROLE_KEY to the client - Client code should only use NEXT_PUBLIC* vars - All server actions that import db client must set `export const runtime = "nodejs"`

package_scripts:
setup: - pnpm dlx shadcn@latest init - pnpm dlx shadcn@latest add button card input label badge avatar dropdown-menu dialog sheet tooltip toast separator tabs
db: - drizzle:generate: drizzle-kit generate - drizzle:migrate: tsx ./src/db/run-migrations.ts - db:seed: tsx ./src/db/seed.ts
dev: - dev: next dev - build: next build - start: next start - typecheck: tsc --noEmit - lint: next lint

conventions:
code_style:
ts: - strict null checks - no unused any - prefer readonly and const
react: - server components by default, client components only when needed - colocate fragments of UI by feature - keep client hooks lightweight
naming: - tables snake_case, columns snake_case - types PascalCase, variables camelCase - events lower_snake with clear verbs, for example claim_attempt
commits: - feat: new feature - fix: bug fix - chore: tooling and deps - refactor: no feature change - db: migrations and schema
pull_requests_checklist: - RLS policies updated for any new tables - Realtime publication updated if streaming is required - Server actions declare runtime nodejs - No service role key in client imports - Added to game registry if feature is a new game - Added tests or manual test notes

drizzle_usage:
when_to_use_drizzle: - Schema definition and migrations - Seeds and data backfills - Server actions that need admin-level writes or cross-table transactions - Derived snapshot writes to room_state with consistency requirements
when_not_to_use_drizzle: - Client-side queries under RLS - Realtime subscriptions - Presence and broadcast
notes: - Use SUPABASE_DB_POOL_URL with prepare=false for runtime - Use DRIZZLE_DATABASE_URL for migrations and seed - Keep migrations idempotent where possible, safe to run in CI - Do not create tables without enabling RLS and policies in the same migration

supabase_js_usage:
client_side: - Auth session, sign in, sign out - RLS-protected selects and inserts by authenticated users - Realtime:
presence: supabase.channel with presence config
broadcast: low latency action lane, small payloads
postgres_changes: stream room_events and select tables per game
server_side: - Reading current user via createServerClient in RSC and actions - Lightweight reads that must honor RLS for the requesting user - Admin tasks should use drizzle with pooled server DB client, not Supabase JS
rules: - Enable WAL for rooms, room_players, room_events, and any game tables that need streaming - Add new tables to `supabase_realtime` publication in a migration

auth_and_rls:
principles: - Everything is private by default - Users can only see rooms that are public or rooms they joined - Only room members can read and write room_events - room_state is readable by members, writable only by service role
checks: - New tables must enable RLS and include select and write policies - Policies must reference auth.uid() where appropriate - Add helpful indices if policies filter by user_id or room_id

state_management:
tanstack_query: - Source of truth for anything that ultimately comes from the DB - Use for room lists, event tails, snapshot loads, profile reads - Disable refetch on focus for high-churn live views when also streaming
zustand_or_jotai: - Use for ephemeral UI and optimistic state - Keep stores per feature to avoid global sprawl - Do not store server data long term in client stores
realtime_flow: - Broadcast an action for immediate UI intent - Persist authoritative event via insert into room_events (client if allowed, server action if validation is required) - Stream INSERT via postgres_changes for reconciliation - Optional snapshot: write summarized state to room_state in a server action

game_module_contract:
descriptor: - slug: unique, stable - name: display name - channelEvents.action: list of allowed broadcast types for the game - load(): dynamic import of Client component
client_component: - props: { roomId: string } - opens presence+broadcast channel "room:<id>", and a postgres_changes channel for room_events - initializes Query cache from room_state then replays live events
db: - Use the shared tables for events and state - Add per-game tables only when necessary, with RLS and publication updates
registry: - src/lib/games/registry.ts is the single place to register a game

room_lifecycle:
lobby: - host creates room, auto joins as host - other users join, role=player or spectator
active: - host starts game, write started_at and status=active - broadcast tick optional, keep server authoritative writes minimal
ended: - write ended_at and status=ended - lock writes except admin post-processing
recovery: - late joiners load room_state then subscribe to room_events tail - clients reconcile divergence with snapshot version

security:
rules: - Never use service role in the browser - Validate user actions on server when they affect scoring or other users - Rate limit sensitive server actions with simple token bucket or time checks - Sanitize user-generated content, limit payload sizes in broadcast
rls_testing: - Write small SQL probes in migration PR to verify policies

performance:
guidelines: - Keep broadcast payloads under a few hundred bytes - Prefer JSON schema that allows partial updates - Paginate room_events and limit live tail to the last N entries - Use memoization and windowing for large lists - Avoid re-subscribing channels on every render

observability:
logging: - Log important server actions with roomId, userId, type - Redact PII and auth tokens
metrics: - Track channel subscribe time, reconnect counts, dropped events
errors: - Surface toast notifications for realtime disconnects and policy denials

testing:
levels: - unit: pure reducers, utilities, validation - component: client components with mocked Supabase client - integration: server actions with an ephemeral database or test schema - e2e: Playwright basic flows, login, create room, join, send action, see event
seeds: - Seed at least two games and a demo public room for manual testing

deployment:
vercel: - Set all env vars in Preview and Production - Never run migrations in Vercel build step
ci: - Run drizzle:migrate on merge to main via GitHub Actions - Optional: run db:seed on non-prod
supabase: - Add vercel domains to Auth redirect allowlist - Confirm WAL enabled for realtime tables

common_tasks:
create_room: - server action uses drizzle to insert rooms and room_players, then redirect
append_event: - if trivial and allowed by RLS, client inserts into room_events - if validation or cross-writes required, call server action using drizzle
snapshot_state: - server action computes snapshot from recent events and writes to room_state with service role
add_game: - add registry entry, create Client.tsx, seed games row, optional tables with RLS, update publication

lint_rules_for_ai:
do: - Prefer server components; use "use client" only when needed - Keep new tables behind RLS with policies in same migration - Use drizzle for schema and server writes that need authority - Use Supabase JS for client reads under RLS and for realtime - Keep actions small, composable, and typed
avoid: - Writing SQL directly into Supabase editor - Importing service role key in client bundles - Mixing game-specific logic into shared shell components - Oversized broadcast messages or per-keystroke DB writes

file_templates:
server_action_header: |
"use server";
export const runtime = "nodejs";
client_component_header: |
"use client";

review_examples:
adding_new_table: - write drizzle schema - generate migration - manual migration to add RLS, policies, indexes, publication - migrate, seed if needed
adding_new_realtime_stream: - ensure WAL is enabled and table is in publication - subscribe via supabase.channel with postgres_changes filter - test insert, verify client receives payload

acceptance_criteria_for_prs:

- App starts without runtime errors
- TypeScript passes
- New migrations apply on a clean database
- RLS policies block non-members for private data
- Realtime subscriptions function for the intended tables
- No leakage of service role key or secrets

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nick-neely) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
