## martin-aia

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent Onboarding System - A monorepo for managing agent recruitment events, tracking invitations, and recording attendance with reward calculations. Three user types: Admin (full system control), Agent (invite new members), and New Member (registers and attends events).

## Commands

### Development
```bash
pnpm dev:admin    # Admin portal at http://localhost:3000
pnpm dev:agent    # Agent portal at http://localhost:3001
pnpm dev:public   # Public pages at http://localhost:3002
```

### Build & Quality
```bash
pnpm build        # Build all apps
pnpm lint         # ESLint across all packages
pnpm -r typecheck # TypeScript validation
```

### Supabase (Local Development)
```bash
npx supabase start           # Start local Supabase stack
npx supabase stop            # Stop local Supabase
npx supabase db reset        # Reset database and run migrations
npx supabase migration new   # Create new migration
```

**Local Supabase Ports:**
- API: 54321
- Database: 54322
- Studio: 54323
- Email testing (Inbucket): 54324

## Architecture

### Monorepo Structure (pnpm workspaces)
```
apps/
├── admin-portal/     # Campaign, agent, tier management, reports
├── agent-portal/     # Agent-facing: view campaigns, manage invitations
└── public-pages/     # No auth: registration, check-in/out via QR

packages/
├── shared-types/     # TypeScript types & enums for database entities
└── shared-ui/        # shadcn/ui components, Supabase client, utilities

supabase/
├── migrations/       # PostgreSQL schema migrations
├── config.toml       # Local Supabase configuration
└── seed.sql          # Seed data
```

### Tech Stack
- **Frontend:** React 18, Vite 5, TypeScript, TanStack Router, TanStack Query
- **Styling:** Tailwind CSS, shadcn/ui (Radix primitives)
- **Forms:** react-hook-form + zod validation
- **Backend:** Supabase (PostgreSQL 15, Auth, RLS)
- **PDF/Charts:** jspdf, html2canvas, recharts, qrcode.react

### Data Flow
Campaigns → Slots (time windows) → Invitations (unique tokens) → Attendance (PIN-based check-in/out) → Rewards

### Key Domain Concepts
- **Tiers:** Agent tiers with reward amounts and invitation limits per slot
- **Capacity Types:** `agent` or `business_partner` (affects rewards)
- **PIN Codes:** Generated per slot, distributed at events, one-time use
- **Invitation Tokens:** UUID v4 one-time links shared via WhatsApp/SMS

## Shared Packages

### @agent-system/shared-types
Import types and enums:
```typescript
import { Campaign, Agent, InvitationStatus, CampaignStatus } from '@agent-system/shared-types';
```

### @agent-system/shared-ui
Import components and utilities:
```typescript
import { Button, Card, Input, Form, Table, Dialog, Select, cn, supabase } from '@agent-system/shared-ui';
```

## Database

### Core Tables
`campaigns`, `slots`, `tiers`, `agents`, `invitations`, `pin_codes`, `attendance`, `rewards`

### RLS Policies
- Admins: Full access via `is_admin()` function (checks `raw_user_meta_data->>'role' = 'admin'`)
- Agents: See only own data via `get_agent_id()` function
- Public (anon): Can read/update PIN codes and insert/update attendance for check-in flow

### Enums (in PostgreSQL and TypeScript)
`invitation_type`, `campaign_status`, `invitation_status`, `capacity_type`, `role_type`, `agent_status`, `reward_status`

### Test Users (Production)
| Email | Password | Role | Access |
|-------|----------|------|--------|
| `admin@test.com` | `@Abc1234` | admin | Full system access via Admin Portal |
| `agent@test.com` | `@Abc1234` | agent | Agent Portal with own data only |

**Agent Test Data:**
- Name: Test Agent
- Code: AGT001
- Tier: Standard Agent ($50 reward, 10 invites/slot)
- Phone: +6591234567
- NRIC: S1234567A

## Deployment

### GitHub Repository
- **Repo:** https://github.com/paul-techiesapp/martin-aia
- **Branch:** `main`
- **Visibility:** Private

### Production URLs (Render Static Sites)
- **Render Account:** `racc.main2026@gmail.com` (workspace: RACC's workspace, `tea-d7os1f7avr4c73cekin0`)

| App | URL | Dashboard |
|-----|-----|-----------|
| Admin Portal | https://racc-admin-portal.onrender.com | [Dashboard](https://dashboard.render.com/static/srv-d87br30g4nts73doju9g) |
| Agent Portal | https://racc-agent-portal.onrender.com | [Dashboard](https://dashboard.render.com/static/srv-d87br60jo6nc738ik270) |
| Public Pages | https://racc-public-pages.onrender.com | [Dashboard](https://dashboard.render.com/static/srv-d87br7b7uimc73c8jn2g) |

**Auto-deploy:** Enabled - pushes to `main` branch trigger automatic deployments.

### Production Supabase
- **Project ID:** `mjtdsevynrtcmafsnxsj`
- **Name:** BOP Website
- **URL:** https://mjtdsevynrtcmafsnxsj.supabase.co
- **Dashboard:** https://supabase.com/dashboard/project/mjtdsevynrtcmafsnxsj
- **Region:** ap-northeast-1

### Build Commands (Render)
Each app uses pnpm workspace filtering:
```bash
pnpm install && pnpm --filter admin-portal build   # Admin
pnpm install && pnpm --filter agent-portal build   # Agent
pnpm install && pnpm --filter public-pages build   # Public
```

## Environment Variables

### Local Development
Each app needs `.env` (copy from `.env.example`):
```
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=<your-anon-key>
```

Get the anon key from `npx supabase status` after starting local Supabase.

### Production (Render)
Environment variables configured in Render dashboard:
```
VITE_SUPABASE_URL=https://mjtdsevynrtcmafsnxsj.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1qdGRzZXZ5bnJ0Y21hZnNueHNqIiwicm9sZSI6ImFub24iLCJpYXQiOjE3Nzc4NjU4NTcsImV4cCI6MjA5MzQ0MTg1N30.7w5WZyHlTJcXImWXtsFa8QinDfiOpt05NnHFoSuGqo0
```

## Patterns

### React Query Hooks
Custom hooks in `src/hooks/` wrap Supabase queries with React Query:
```typescript
// Example: useCampaigns(), useAgents(), useSlots(), useTiers()
const { data, isLoading } = useCampaigns();
const mutation = useCreateCampaign();
```

### Form Handling
```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Form, FormField } from '@agent-system/shared-ui';
```

### Path Aliases
All apps use `@/*` → `./src/*` for imports.

---
> Source: [paul-techiesapp/martin-aia](https://github.com/paul-techiesapp/martin-aia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
