## ai-app-generator

> An AI-powered platform that lets non-technical users build complete web apps through natural language + visual configuration. Apps with backend needs are powered by Supabase out-of-the-box.

# AI App Generator - Project Architecture

## Overview
An AI-powered platform that lets non-technical users build complete web apps through natural language + visual configuration. Apps with backend needs are powered by Supabase out-of-the-box.

## Target User
People with zero coding knowledge who want to turn their app idea into a real, deployed web application quickly.

## Core Principles
1. **Speed over flexibility** - Get a working app in minutes, not hours
2. **Visual-first** - Users configure through UI, not code
3. **Modular** - Build like LEGO: pick features, combine them
4. **Template-rich** - Start from proven templates, customize what matters
5. **User sovereignty over UI** - Templates handle structure; users control look/feel via visual editor

## Architecture

```
User Idea
    ↓
Platform (Next.js 16 + React 19)
  ├── Template Selection
  ├── Natural Language Input
  ├── Module Builder (LEGO blocks)
  └── Visual Preview/Editor
    ↓
Engine (AI Code Generation)
  ├── Prompt Assembler (template + modules + user intent)
  ├── Claude API Call
  └── Code Output → generated/[app-id]/
    ↓
Deployer (Vercel-first)
  ├── Build & Deploy
  └── Live URL
    ↓
Supabase Backend (auto-provisioned)
  ├── Auth
  ├── Database (PostgreSQL)
  ├── Storage
  ├── Edge Functions
  └── Realtime
```

## Directory Structure

```
ai-app-generator/
├── apps/platform/        # Main Next.js app (UI + API routes)
│   ├── app/
│   │   ├── page.tsx          # Landing
│   │   ├── create/           # Multi-step creation flow
│   │   ├── apps/             # My apps dashboard
│   │   └── api/              # Backend API routes
│   ├── lib/
│   │   ├── supabase.ts       # Supabase admin client
│   │   └── database.types.ts # DB types
│   └── prisma/schema.prisma  # Platform DB schema
├── engine/               # AI code generation engine
│   ├── prompt-builder.ts     # Assembles prompts from config
│   ├── module-registry.ts    # Available feature modules
│   └── generators/           # Per-framework generators
├── templates/            # App templates
│   └── [template-id]/
│       ├── metadata.json
│       ├── prompt.md
│       └── preview.png
├── generated/            # Output directory for generated apps
│   └── [app-id]/
└── deployer/             # Deployment scripts
    └── vercel.ts
```

## Database Schema (Platform)

```prisma
model App {
  id          String   @id @default(cuid())
  name        String
  description String?
  prompt      String   // User's natural language description
  template    String   // Template ID used
  modules     String[] // Selected module IDs
  status      String   @default("draft") // draft | generating | ready | deployed | error
  port        Int?
  url         String?
  error       String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

## Module System

Modules are self-contained feature blocks that the engine knows how to generate:

| Module ID | Feature | Supabase Service |
|-----------|---------|------------------|
| `auth` | Login/signup, user profiles | Auth |
| `database` | CRUD tables, relationships | Database |
| `storage` | File upload/download | Storage |
| `realtime` | Live updates, chat | Realtime |
| `edge-functions` | Custom serverless logic | Edge Functions |
| `admin` | Admin dashboard | - |
| `payments` | Stripe integration | - |
| `notifications` | Push/email notifications | - |

Each module has:
- `metadata.json` - name, description, icon, category
- `prompt.md` - How to tell Claude to generate this feature
- `schema.md` - Supabase schema requirements (tables, RLS policies, etc.)
- `dependencies` - Other modules this one needs

## Template System

Templates are complete app starting points:

| Template ID | Description | Modules Included |
|-------------|-------------|------------------|
| `blank` | Empty Next.js app | none |
| `blog` | Content blog | auth, database, storage |
| `ecommerce` | Online store | auth, database, storage, payments |
| `saas` | SaaS dashboard | auth, database, admin, payments |
| `social` | Social network | auth, database, storage, realtime |
| `marketplace` | Service marketplace | auth, database, storage, payments |

## Generation Flow

1. User selects template (or blank) + describes their app
2. User toggles/modules modules via visual builder
3. System assembles a detailed prompt:
   - Base: "Build a Next.js 16 app using React 19, TypeScript, Tailwind CSS 4, shadcn/ui"
   - Template context: "Start from the [template] structure..."
   - Module prompts: "Add authentication using Supabase Auth with email/password and OAuth..."
   - Schema: "Create these Supabase tables with RLS policies..."
   - User intent: "The user specifically wants: [description]"
4. Claude generates complete project code
5. Code saved to `generated/[app-id]/`
6. Deployer pushes to Vercel
7. Supabase project created/configured via API

## Tech Stack

- **Frontend**: Next.js 16, React 19, TypeScript, Tailwind CSS 4, shadcn/ui
- **Backend**: Supabase (PostgreSQL, Auth, Storage, Realtime, Edge Functions)
- **ORM**: Prisma 7 (for platform DB only; generated apps use Supabase client directly)
- **AI**: Anthropic SDK (Claude Sonnet 4.6 / Opus 4.7)
- **Deployment**: Vercel (for generated apps)

---
> Source: [3760362028-hash/ai-app-generator](https://github.com/3760362028-hash/ai-app-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
