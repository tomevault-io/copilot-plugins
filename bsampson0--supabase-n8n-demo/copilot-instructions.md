## supabase-n8n-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DevOps Hub is a deployment monitoring and automation demo showcasing integration between Supabase (authentication + database + realtime) and n8n workflow automation. The architecture is a monorepo with three main components:

- **client/**: React + Vite frontend with Tailwind CSS
- **server/**: Express backend providing SSE streaming and webhook endpoints
- **supabase/**: Database migrations for the deployments table

## Commands

### Development
```bash
# Install all dependencies (client, server, and root)
npm run install:all

# Run both client (port 3000) and server (port 3001) concurrently
npm run dev

# Run client only
npm run dev --prefix client

# Run server only
npm run dev --prefix server
```

### Client (Vite + React)
```bash
cd client
npm run dev      # Start Vite dev server on port 3000
npm run build    # Build for production (runs TypeScript compiler first)
npm run preview  # Preview production build
```

### Server (Express + TypeScript)
```bash
cd server
npm run dev      # Start with tsx watch (hot reload)
npm run build    # Compile TypeScript to dist/
npm run start    # Run compiled JavaScript from dist/
```

### Supabase CLI Workflow
```bash
# First time setup
npm run supabase:init   # Initialize supabase/ directory (if no config.toml)
npm run supabase:login  # Authenticate CLI (or set SUPABASE_ACCESS_TOKEN)
npm run supabase:link   # Link to your Supabase project

# Migrations
npm run db:push         # Apply migrations to linked project
npm run db:new -- <name>  # Create new migration file

# Alternative: run supabase/deployments.sql directly in Supabase SQL Editor
```

## Architecture

### Authentication Flow
1. **AuthContext** (`client/src/context/AuthContext.tsx`): React context that wraps the app, manages session state via `supabase.auth.onAuthStateChange`, and provides `user`, `session`, `loading`, and `signOut` to components.
2. **Protected/Public Routes** (`client/src/App.tsx`): `ProtectedRoute` redirects to `/auth` if no session; `PublicRoute` redirects to `/dashboard` if already authenticated.
3. **Server Middleware** (`server/src/middleware/auth.ts`): Validates JWT from `Authorization: Bearer <token>` header using Supabase service role key. Currently defined but not actively used on webhook routes.

### Data Flow: Deployment Trigger → n8n Webhook
This is the core integration pattern of the application:

1. **User triggers deployment** in Dashboard (`client/src/pages/Dashboard.tsx`)
   - Form collects `service`, `version`, `environment`
   - Client calls `supabase.from('deployments').insert([row])`

2. **Supabase Database Webhook** (configured in Supabase Dashboard)
   - Listens for INSERT events on `public.deployments` table
   - Sends POST request to n8n workflow URL

3. **n8n Workflow** (external, not in this repo)
   - Receives deployment event from Supabase
   - Processes automation (e.g., Slack notification, update monitoring)
   - Sends callback POST to `http://localhost:3001/api/webhook/n8n`

4. **Express Server** (`server/src/routes/webhook.ts`)
   - `/api/webhook/n8n`: Receives n8n callback, broadcasts to all SSE clients
   - `/api/events/stream`: SSE endpoint where Dashboard connects for real-time n8n events

5. **Dashboard Updates** in two ways:
   - **Supabase Realtime**: New deployment rows appear immediately via `supabase.channel('deployments-realtime')`
   - **SSE Stream**: n8n callbacks show in Event Log via EventSource connection to `/api/events/stream`

### Repository Monitoring Workflow (GitHub + Netlify → n8n)

The application includes a comprehensive monitoring system that tracks GitHub repository activity and Netlify deployments:

**n8n Workflow** (`n8n-workflows/repo-monitoring-workflow.json`):
- Two webhook triggers: one for GitHub events, one for Netlify deploy notifications
- Processes four event types:
  - `github_push`: Git commits and pushes
  - `github_pr`: Pull request creation, updates, merges
  - `github_issue`: Issue creation and updates
  - `netlify_deploy`: Build started, succeeded, failed events
- Each event is:
  1. Transformed into standardized format
  2. Inserted into `event_log` table (see migration `202604100002_create_event_log.sql`)
  3. Inserted into `deployments` table (for Netlify events only, when build completes)
  4. Broadcast to Express backend via `/api/webhook/n8n` (appears in Dashboard Event Log via SSE)
  5. Sent to Slack/Discord for team notifications

**Setup**: See `SETUP_MONITORING.md` for complete configuration guide including:
- GitHub webhook configuration (Settings → Webhooks)
- Netlify deploy notifications (Site Settings → Deploy notifications)
- n8n credential setup (Supabase, Slack, Discord)
- Testing the complete event flow

**Event Log Table** (`public.event_log`):
- Stores all GitHub/Netlify events with columns for repo_name, branch, commit_sha, pr_number, deploy_id, etc.
- Full webhook payload stored in `payload` jsonb column for debugging
- RLS: authenticated users can read, service_role can insert
- Enabled for Realtime subscriptions

This monitoring workflow runs in parallel to the manual deployment trigger workflow, providing comprehensive visibility into repository and deployment activity.

### Vite Proxy Configuration
The client dev server (`vite.config.ts`) proxies `/api/*` requests to `http://localhost:3001` so frontend code can use relative URLs like `/api/events/stream` without CORS issues during development.

### Database Schema

**Deployments Table** (`supabase/migrations/202604100001_create_deployments.sql`):
- **Table**: `public.deployments` with columns `id`, `service`, `version`, `environment`, `status`, `triggered_by`, `deployed_at`
- **RLS Policies**: Authenticated users can SELECT and INSERT
- **Realtime**: Table added to `supabase_realtime` publication for live subscriptions
- **Seed Data**: Three sample rows inserted if table is newly created

**Event Log Table** (`supabase/migrations/202604100002_create_event_log.sql`):
- **Table**: `public.event_log` stores all GitHub and Netlify monitoring events
- **Columns**: `event_type`, `event_source`, `event_action`, `repo_name`, `branch`, `commit_sha`, `pr_number`, `deploy_id`, `payload` (jsonb), etc.
- **RLS Policies**: Authenticated users can SELECT, service_role can INSERT
- **Realtime**: Enabled for live event streaming to Dashboard
- **Indexes**: On `created_at`, `event_type`, and `repo_name` for efficient querying

### Environment Variables

**Client** (`client/.env`):
```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
```

**Server** (`server/.env`):
```env
SUPABASE_URL=your-supabase-url
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
PORT=3001
```

If client env vars are missing, the app displays a `MissingConfigScreen` with setup instructions instead of attempting to initialize Supabase.

## Key Files

- `client/src/lib/supabase.ts`: Supabase client initialization, exports `supabase` instance and `Deployment` type
- `client/src/context/AuthContext.tsx`: Authentication state management
- `client/src/pages/Dashboard.tsx`: Main UI with deployment table, trigger form, Realtime subscription, and SSE event log
- `server/src/routes/webhook.ts`: SSE streaming and n8n webhook receiver for GitHub/Netlify/deployment events
- `server/src/middleware/auth.ts`: JWT validation middleware (not currently applied to routes)
- `n8n-workflows/repo-monitoring-workflow.json`: Complete n8n workflow for monitoring GitHub and Netlify events
- `supabase/migrations/202604100001_create_deployments.sql`: Deployments table schema
- `supabase/migrations/202604100002_create_event_log.sql`: Event log table for monitoring events
- `SETUP_MONITORING.md`: Comprehensive guide for configuring GitHub/Netlify webhooks and n8n workflow

## Testing the Integration

### Manual Deployment Trigger
1. Ensure Supabase tables exist: `npm run db:push`
2. Start both servers: `npm run dev`
3. Sign up/login at `http://localhost:3000`
4. Trigger a deployment from Dashboard form
5. Check Event Log for Realtime insertion event
6. (If n8n webhook configured) n8n callback should appear in Event Log via SSE

### Repository Monitoring Workflow
1. Follow `SETUP_MONITORING.md` to configure GitHub/Netlify webhooks and n8n
2. Test GitHub events:
   - Push a commit → check n8n executions and `event_log` table
   - Open a PR → verify event appears in Dashboard
   - Create an issue → check notifications sent to Slack/Discord
3. Test Netlify deployments:
   - Trigger a Netlify build (push to main or manual deploy)
   - Verify "building" event appears immediately
   - Check "ready" or "failed" event creates row in both `event_log` and `deployments` tables
4. Verify complete data flow: GitHub → n8n → Supabase → Dashboard (via Realtime and SSE)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bSampson0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
