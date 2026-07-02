## lucy

> ﻿# CLAUDE.md -- Lucy AI Platform

﻿# CLAUDE.md -- Lucy AI Platform

## Project Overview

Lucy is a multi-provider AI chat platform with a visual workflow builder, dual storage system, and project integration framework. It is built for internal teams and company onboarding -- allowing users to chat with OpenAI, Anthropic, Google, or local (Ollama / LM Studio) models through a unified interface, build AI pipelines with drag-and-drop workflows, and connect Lucy to external apps (like Contractors Room) so the AI has live business context.

Key capabilities:
- **Multi-provider chat** -- stream responses from GPT-4o, Claude, Gemini, or local models (Ollama/LM Studio) through a single UI
- **AI personas** -- 5 built-in system-prompt personas + create custom ones; selected via a chip in the chat input bar
- **Message editing and regeneration** -- inline edit any user message; regenerate any assistant reply
- **Token tracking** -- per-message and per-conversation token estimates shown in the UI
- **Visual workflow builder** -- drag-and-drop node graph for building multi-step AI pipelines
- **Dual storage** -- works standalone with localStorage or connected to Supabase PostgreSQL
- **Integration system** -- external apps register their schema/actions so Lucy can read their data and act on it
- **Auth system** -- Supabase Auth with email/password, Google OAuth, and route-level middleware
- **Embeddable widget** -- one-line script tag to embed Lucy chat in any web app
- **PWA** -- Web App Manifest + three edge-generated icon sizes (32, 192, 512) for installability
- **Docker** -- multi-stage Dockerfile + docker-compose for production container deployment
- **Test suite** -- 219 tests across 33 suites (Jest + React Testing Library)

## Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| Next.js | 16.2.9 | App Router framework (Turbopack), API routes, SSR |
| React | 19.2.7 | UI rendering |
| TypeScript | 5.x | Type safety across the entire codebase |
| Tailwind CSS | 3.4.15 | Utility-first styling, dark/light mode via `class` strategy |
| Zustand | 5.0.2 | Lightweight state management (4 stores: chat, conversations, settings, personas) |
| @xyflow/react | 12.11.0 | React Flow v12 for the workflow canvas |
| Supabase | 2.108.1 | Optional PostgreSQL backend + Auth + RLS |
| OpenAI SDK | 6.42.0 | OpenAI API client (also used for Ollama/LM Studio local endpoints) |
| Anthropic SDK | 0.104.1 | Anthropic API client |
| Google AI SDK | 0.24.1 | Gemini API client |
| react-markdown | 10.1.0 | Markdown rendering in chat messages |
| rehype-highlight | 7.0.0 | Syntax highlighting in code blocks |
| remark-gfm | 4.0.0 | GitHub Flavored Markdown |
| lucide-react | 1.17.0 | Icon library (brand icons removed in 1.x) |
| Jest | 30.4.2 | Test runner (via `next/jest` SWC transform) |
| ESLint | 9.x | Linting via flat config (`eslint.config.mjs`) + eslint-config-next 16 |
| React Testing Library | 16.1.0 | Component testing utilities |
| @testing-library/jest-dom | 6.6.3 | Custom DOM matchers |
| @testing-library/user-event | 14.5.2 | User interaction simulation |

## Architecture

### High-Level Diagram

```
+------------------------------------------------------------------+
|                         Next.js App Router                        |
|                                                                   |
|  app/                                                             |
|  +-- layout.tsx          AuthProvider > StorageProvider > StoreSync > ThemeProvider
|  +-- chat/page.tsx       Main chat UI                             |
|  +-- personas/page.tsx   Persona management                       |
|  +-- workflows/          Workflow list + editor                   |
|  +-- settings/           API keys, local models, theme, data      |
|  +-- auth/               login, signup, forgot-password, callback |
|  +-- api/chat/route.ts   SSE streaming endpoint (rate-limited)    |
|  +-- api/models/route.ts Model list + optional local discovery    |
|  +-- api/embed/route.ts  Embeddable widget script                 |
+------------------------------------------------------------------+
         |                    |                    |
         v                    v                    v
+----------------+  +------------------+  +------------------+
| lib/providers/ |  | lib/storage/     |  | lib/integrations/|
| OpenAI         |  | StorageAdapter   |  | Registry         |
| Anthropic      |  | LocalStorage     |  | Context Builder  |
| Gemini         |  | Supabase         |  | Action Executor  |
| LocalProvider  |  +------------------+  +------------------+
+----------------+           |                    |
         |                   v                    v
         v          +------------------+  +------------------+
+----------------+  | lib/supabase/    |  | lib/workflow/    |
| lib/store/     |  | client.ts        |  | engine.ts        |
| chat.ts        |  | schema.sql       |  | store.ts         |
| conversations  |  | auth.tsx         |  | registry.ts      |
| settings.ts    |  +------------------+  +------------------+
| personas.ts    |
+----------------+

proxy.ts -- route protection (redirects unauthenticated users when Supabase is enabled)
```

### Data Flow: Chat Message

```
User types message
  -> ChatPage.handleSend()
  -> useChatStore (set loading/streaming state)
  -> useConversationsStore.addMessageToConversation() (persist user msg)
  -> fetch POST /api/chat (with API key in header, or ollamaUrl/lmStudioUrl for local)
     -> proxy.ts checks auth (if Supabase enabled)
     -> API route checks rate limit (30 req/min per IP)
     -> API route reads provider, resolves API key
     -> For 'local' provider: uses LocalProvider with custom baseURL, no key required
     -> Active persona system prompt prepended to messages (if persona selected)
     -> Optionally injects project context into system prompt
     -> Calls provider.chat() with streaming callback
     -> Streams SSE events: data: {"content":"..."}\n\n
  -> Client parseSSEStream() calls appendStreamingContent() per chunk
  -> On [DONE]: save full assistant message, clear streaming state
```

### Auth Flow

```
Supabase configured?
  No  -> Standalone mode, all routes public, user is anonymous
  Yes -> Auth enabled
           Login page -> Supabase signInWithPassword() or signInWithOAuth(google)
           OAuth -> browser redirected to Google -> back to /auth/callback
           /auth/callback -> exchanges code for session, redirects to /chat
           proxy.ts -> verifies session cookie on protected routes
           Header shows user avatar + sign-out dropdown

Public paths (never blocked by middleware):
  /auth/*, /api/*, /onboarding, /embed, /personas, /_next/, /favicon
Protected paths (redirect to /auth/login if no session):
  /chat, /workflows, /settings
```

### Storage Flow

```
Zustand Store <--write-through--> StorageAdapter interface
                                       |
                    +------------------+------------------+
                    |                                     |
             LocalStorageAdapter                 SupabaseStorageAdapter
             (browser localStorage)              (Supabase PostgreSQL)
```

`StorageProvider` detects at runtime whether `NEXT_PUBLIC_SUPABASE_URL` is set. `StoreSync` bootstraps stores from the adapter on mount.

### Local Provider Flow

```
ModelSelector mounts
  -> fetch GET /api/models?includeLocal=true (AbortSignal timeout 4s)
     -> server calls discoverLocalModels(ollamaUrl, lmStudioUrl)
        -> GET localhost:11434/v1/models (Ollama)
        -> GET localhost:1234/v1/models  (LM Studio)
        -> either/both can fail silently
     -> returns enriched model list + localStatus
  -> if any local server available: setLocalModels(), show Local optgroup
  -> if nothing available: show "No local models (start Ollama)" disabled option

User picks a local model and sends message:
  -> fetch POST /api/chat with headers: x-ollama-url, x-lmstudio-url
     -> chat route detects provider === 'local', skips API key check
     -> LocalProvider.chat() -> new OpenAI({ baseURL: ollamaUrl/v1 })
     -> streams response back as SSE
```

### Integration Flow

```
registerProject() at startup
  -> Integration added to in-memory registry (Map)
  -> /api/chat checks if projectId is provided
  -> buildProjectContext() queries registered tables via Supabase
  -> Context text prepended to system prompt
  -> AI responds with awareness of user's live data
```

### App shell & Settings/Admin IA

Design spec: `docs/superpowers/specs/2026-06-09-settings-admin-appshell-design.md`. Implementation plan: `docs/superpowers/plans/2026-06-09-settings-admin-appshell.md`.

**App shell** — `components/layout/AppShell.tsx` composes `Sidebar` + `Topbar` and wraps every authenticated app page. Auth pages (`/auth/*`) and the embed page (`/embed`) are **not** wrapped — they render standalone.

**Navigation** — the Sidebar provides top-level nav: Chat, Personas, Workflows, Connectors, Settings, and Admin. Admin is gated: visible only when `useIsAdmin()` returns true (calls `GET /api/admin/me` which checks the `lucy_role` admin flag in Supabase auth `app_metadata` via `lib/auth/admin.ts`). Non-admins see the Admin item greyed and locked. Roles are managed in the Admin panel's "Users & roles" card (`POST /api/admin/roles`, service-role only — users cannot self-promote). On a fresh deployment, the oldest account is auto-promoted to admin (or a `LUCY_ADMIN_EMAIL` match, legacy bootstrap only).

**Settings** — `app/settings/layout.tsx` renders a `SettingsNav` sidebar alongside the active sub-route. Settings holds ONLY app/product configuration (no personal items). Sub-routes:
- `general` — five-theme picker (swatch cards) + data controls
- `providers` — API keys (OpenAI/Anthropic/Google/etc.) + local model URLs (Ollama/LM Studio)
- `memory` — user memory controls (`MemoryPanel`)
- `voice` — STT/TTS configuration
- `api-access` — outbound Lucy API key management
- Legacy redirects: `profile` → `/account/profile`; `account`/`security` → `/account/security`; `integrations` → `/connectors`

**Account** — `app/account/layout.tsx` renders an `AccountNav` sidebar (Profile · Security · Billing); reached via the Topbar avatar menu ("Account"). `/account` redirects to `/account/profile`. Personal concerns live here: `profile` (display name/avatar/company → `user_profiles`), `security` (password, TOTP + email 2FA, devices), `billing` (scaffold: plan + usage, no payment integration yet). `/account` is in `proxy.ts` `protectedPrefixes`.

**Admin** — `app/admin/page.tsx` renders `AdminMemoryPanel` (embedder config, retention policy, bulk deletion) and a storage-mode indicator. Access is gated server-side via `useIsAdmin`.

**Connectors** — `app/connectors/page.tsx` holds the integration management UI (previously at `settings/integrations`). The old URL redirects.

## Project Structure

```
C:\RepositoryAI\LucyAI\
|
+-- app/                          # Next.js App Router pages
|   +-- layout.tsx                # Root layout: inline theme script + AuthProvider > StorageProvider > StoreSync > ThemeProvider
|   +-- page.tsx                  # Public landing page (modern default; ?v=corporate light version; components/landing/)
|   +-- icon.tsx                  # Dynamic favicon 32x32 (purple "L", edge runtime, next/og)
|   +-- icon-192.tsx              # PWA icon 192x192 (edge runtime, next/og)
|   +-- icon-512.tsx              # PWA icon 512x512 (edge runtime, next/og)
|   +-- manifest.ts               # Web App Manifest: name, start_url, icons, theme_color
|   +-- error.tsx                 # React error boundary -- "Something went wrong" + Try again
|   +-- not-found.tsx             # 404 page with link back to chat
|   +-- loading.tsx               # Global loading fallback (animated bouncing dots)
|   +-- globals.css               # Global styles + Tailwind imports
|   +-- chat/
|   |   +-- page.tsx              # Main chat page (sidebar + window + input + keyboard shortcuts)
|   |   +-- loading.tsx           # Chat skeleton loader (header + sidebar + message area)
|   +-- personas/
|   |   +-- page.tsx              # Persona management (view, create, edit, delete, set active)
|   +-- workflows/
|   |   +-- page.tsx              # Workflow list (grid of cards)
|   |   +-- loading.tsx           # Workflows skeleton loader
|   |   +-- [id]/page.tsx         # Workflow editor (canvas + toolbar + panels)
|   +-- admin/
|   |   +-- page.tsx              # Admin panel: AdminMemoryPanel + user roles + storage-mode indicator (gated by lucy_role)
|   +-- connectors/
|   |   +-- page.tsx              # Connector / integration management UI (moved from settings/integrations)
|   +-- account/
|   |   +-- layout.tsx            # Account shell: AccountNav sidebar (Profile/Security/Billing)
|   |   +-- page.tsx              # Redirect → /account/profile
|   |   +-- profile/page.tsx      # Display name, avatar URL, company (user_profiles)
|   |   +-- security/page.tsx     # Password, TOTP + email 2FA, devices & sessions
|   |   +-- billing/page.tsx      # Billing scaffold: plan + usage (no payments yet)
|   +-- settings/
|   |   +-- layout.tsx            # Settings shell: SettingsNav sidebar + <Outlet>
|   |   +-- page.tsx              # Redirect → /settings/providers
|   |   +-- general/page.tsx      # Five-theme picker (swatches) + data controls
|   |   +-- profile/page.tsx      # Redirect → /account/profile
|   |   +-- security/page.tsx     # Redirect → /account/security
|   |   +-- account/page.tsx      # Redirect → /account/security
|   |   +-- providers/page.tsx    # API keys + local models (Ollama/LM Studio)
|   |   +-- memory/page.tsx       # User memory controls (MemoryPanel)
|   |   +-- voice/page.tsx        # STT/TTS configuration
|   |   +-- api-access/page.tsx   # Outbound API key management
|   |   +-- integrations/page.tsx # Redirect → /connectors
|   +-- onboarding/
|   |   +-- page.tsx              # Onboarding wizard
|   +-- embed/
|   |   +-- page.tsx              # Standalone embed page (loaded in iframe)
|   +-- auth/
|   |   +-- login/page.tsx        # Sign-in (email + Google OAuth)
|   |   +-- signup/page.tsx       # Account creation
|   |   +-- forgot-password/page.tsx  # Password reset request
|   |   +-- callback/route.ts     # Supabase OAuth callback handler
|   +-- api/
|       +-- chat/route.ts         # POST - SSE streaming chat, rate-limited 30/min/IP
|       +-- models/route.ts       # GET - list models; ?includeLocal=true probes local servers
|       +-- embed/route.ts        # GET - serves embed script
|       +-- keys/route.ts         # GET/POST/DELETE - API key management (Supabase Auth)
|       +-- screening/
|           +-- start/route.ts    # POST - start a screening (API key auth)
|           +-- [id]/route.ts     # GET/POST - get screening / submit answers
|           +-- route.ts          # GET - list screenings with filters
|
+-- components/                   # React components (all 'use client')
|   +-- ThemeProvider.tsx         # Syncs <html> class with Zustand theme (no-flash approach)
|   +-- chat/
|   |   +-- ChatWindow.tsx        # Message list + token counter header + ExportMenu (Markdown/JSON)
|   |   +-- ChatInput.tsx         # Textarea + PersonaSelector + ModelSelector + send/stop
|   |   +-- ChatMessage.tsx       # Single message bubble: markdown, code blocks (lang+copy+line numbers), edit, regenerate, token count
|   |   +-- ChatSidebar.tsx       # Conversation list: search, date groups, mobile overlay, swipe-to-close, a11y ARIA
|   |   +-- ModelSelector.tsx     # Provider/model dropdown with local model detection
|   |   +-- PersonaSelector.tsx   # Persona chip + listbox dropdown in chat input bar
|   +-- workflow/
|   |   +-- WorkflowCanvas.tsx    # React Flow canvas with drag-drop
|   |   +-- WorkflowToolbar.tsx   # Save/run/name controls
|   |   +-- NodePanel.tsx         # Left sidebar: draggable node types
|   |   +-- NodeConfigPanel.tsx   # Right sidebar: selected node config
|   |   +-- RunPanel.tsx          # Bottom panel: execution logs + output
|   |   +-- nodes/                # Custom React Flow node components
|   |       +-- BaseNode.tsx      # Shared node shell (handles, status badge)
|   |       +-- StartNode.tsx
|   |       +-- LLMNode.tsx
|   |       +-- ConditionNode.tsx
|   |       +-- KnowledgeBaseNode.tsx
|   |       +-- OutputNode.tsx
|   |       +-- TransformNode.tsx
|   |       +-- HttpNode.tsx
|   |       +-- IntegrationNode.tsx
|   +-- embed/
|   |   +-- LucyWidget.tsx        # Embeddable floating chat widget
|   +-- layout/
|   |   +-- AppShell.tsx          # App shell: wraps authenticated pages with Sidebar + Topbar
|   |   +-- Sidebar.tsx           # Left navigation sidebar (Chat, Personas, Workflows, Connectors, Settings, Admin)
|   |   +-- Topbar.tsx            # Top bar for app pages: page title, user avatar + sign-out
|   +-- onboarding/
|   |   +-- OnboardingWizard.tsx  # Step-by-step setup wizard
|   +-- settings/                 # Settings page components
|   |   +-- SettingsNav.tsx       # Left nav for settings sub-routes
|   |   +-- MemoryPanel.tsx       # User memory controls (settings/memory)
|   |   +-- AdminMemoryPanel.tsx  # Admin memory management (embedder, policy, deletion) -- used by /admin
|   |   +-- ProvidersSection.tsx  # API keys + local model URLs
|   |   +-- ApiKeysSection.tsx    # Outbound API key management
|   |   +-- LocalModelsSection.tsx # Ollama / LM Studio URL inputs
|   +-- ui/                       # Reusable primitives
|       +-- Button.tsx
|       +-- Input.tsx
|       +-- Card.tsx
|       +-- Badge.tsx
|       +-- Avatar.tsx
|       +-- Spinner.tsx           # Includes TypingIndicator (three animated dots)
|
+-- lib/                          # Business logic (no UI)
|   +-- providers/                # AI provider abstraction
|   |   +-- types.ts              # AIProvider interface, AIModel, ALL_MODELS, ProviderName (includes 'local')
|   |   +-- index.ts              # getProvider(), getModelById(), getAllModels(), getModelsByProvider(), setLocalModels()
|   |   +-- openai.ts             # OpenAI streaming implementation
|   |   +-- anthropic.ts          # Anthropic streaming implementation
|   |   +-- gemini.ts             # Google Gemini streaming implementation
|   |   +-- local.ts              # LocalProvider: Ollama + LM Studio via OpenAI-compat API + discoverLocalModels()
|   +-- storage/                  # Storage adapter pattern
|   |   +-- index.ts              # StorageAdapter interface + data types
|   |   +-- local.ts              # LocalStorageAdapter implementation
|   |   +-- supabase.ts           # SupabaseStorageAdapter implementation
|   |   +-- provider.tsx          # React context: StorageProvider, useStorage()
|   +-- store/                    # Zustand state stores
|   |   +-- chat.ts               # Ephemeral chat UI state (model, streaming, error)
|   |   +-- conversations.ts      # Conversation list + messages (persisted)
|   |   +-- settings.ts           # API keys + preferences + ollamaUrl + lmStudioUrl (persisted)
|   |   +-- personas.ts           # AI personas: 5 built-ins + custom; activePersonaId; persisted to localStorage
|   |   +-- StoreSync.tsx         # Bootstrap component: loads stores from adapter
|   +-- integrations/             # External app integration system
|   |   +-- registry.ts           # registerProject(), getProject(), in-memory Map
|   |   +-- contractors-room.ts   # Built-in integration definition
|   |   +-- context.ts            # buildProjectContext() for AI system prompts
|   |   +-- actions.ts            # executeAction() dispatcher (insert/update/api/workflow)
|   +-- workflow/                 # Workflow engine and types
|   |   +-- types.ts              # Node types, configs, Workflow, ExecutionResult
|   |   +-- engine.ts             # WorkflowEngine: topological graph execution
|   |   +-- store.ts              # Zustand store for workflow editor state
|   |   +-- registry.ts           # Node type metadata (colors, icons, groups)
|   |   +-- storage.ts            # LocalWorkflowStorage (localStorage persistence)
|   +-- auth/                     # API key authentication
|   |   +-- api-keys.ts           # generateKey, hashKey, validateApiKey, createApiKey, listApiKeys, revokeApiKey, deleteApiKey
|   +-- screening/                # AI screening engine
|   |   +-- index.ts              # startScreening, submitAnswers, getScreening, listScreenings, gradeScreening
|   |   +-- types.ts              # Screening, ScreeningGrade, ContractorProfile, GRADE_LABELS
|   |   +-- grading.ts            # LLM prompt builders + response parsers for screening
|   +-- mcp/                      # Model Context Protocol server
|   |   +-- server.ts             # Standalone MCP server (5 tools, stdio transport)
|   +-- scripts/                  # CLI utilities
|   |   +-- seed-admin-key.ts     # Generates API key for admin@contractorsroom.com
|   +-- supabase/                 # Supabase client + schema
|   |   +-- client.ts             # Singleton browser client, isSupabaseEnabled(); db: { schema: 'lucy' }
|   |   +-- auth.tsx              # AuthProvider context (signIn, signUp, signOut, signInWithGoogle, resetPassword)
|   |   +-- schema.sql            # Full database schema (lucy schema) + RLS + indexes
|   |   +-- api_keys.sql          # API key table migration
|   |   +-- screening_rls_fix.sql # Multi-tenancy RLS fix (created_by column)
|   +-- utils/                    # Shared utilities
|       +-- stream.ts             # parseSSEStream(), createSSEEncoder()
|       +-- markdown.ts           # generateConversationTitle(), hasMarkdown(), truncate()
|       +-- tokens.ts             # estimateTokens(), estimateConversationTokens()
|
+-- __tests__/                    # Jest tests (219 tests, 33 suites)
|   +-- components/
|   |   +-- chat/
|   |   |   +-- ModelSelector.test.tsx
|   |   +-- ui/
|   |       +-- Button.test.tsx
|   +-- lib/
|       +-- integrations/
|       |   +-- registry.test.ts
|       +-- providers/
|       |   +-- index.test.ts
|       +-- storage/
|       |   +-- local.test.ts
|       +-- utils/
|       |   +-- markdown.test.ts
|       +-- workflow/
|           +-- engine.test.ts
|
+-- proxy.ts                 # Route protection (redirects to /auth/login when Supabase is enabled)
+-- jest.config.ts                # Jest config: next/jest (SWC), jsdom environment, @/ module alias
+-- jest.setup.ts                 # Jest setup: loads @testing-library/jest-dom
+-- Dockerfile                    # Multi-stage production build (node:20-alpine)
+-- docker-compose.yml            # docker-compose: port 3000, env passthrough, host.docker.internal for Ollama
+-- .dockerignore                 # Excludes node_modules, .next, .git, .env.local, __tests__
+-- .env.example                  # Environment variable template
+-- package.json
+-- tsconfig.json
+-- tailwind.config.ts            # Custom "lucy" color palette, animations
+-- postcss.config.js
+-- next.config.js
+-- .eslintrc.json
+-- .gitignore
```

### Naming Conventions

- **Files**: kebab-case for lib files (`contractors-room.ts`), PascalCase for components (`ChatWindow.tsx`)
- **Components**: PascalCase export matching filename (`export function ChatWindow`)
- **Types/Interfaces**: PascalCase (`StorageAdapter`, `AIProvider`, `WorkflowNode`)
- **Zustand stores**: `use[Name]Store` pattern (`useChatStore`, `useConversationsStore`, `usePersonasStore`)
- **Supabase tables**: all in the `lucy` schema (no prefix needed — schema provides namespacing)
- **CSS**: Tailwind utility classes only; custom colors under `lucy` namespace
- **Node types**: camelCase identifiers (`knowledgeBase`, `llm`); PascalCase components (`KnowledgeBaseNode`)
- **Test files**: mirror source path under `__tests__/` (e.g., `lib/utils/tokens.ts` -> `__tests__/lib/utils/tokens.test.ts`)

### Where to Put New Features

| What you are adding | Where it goes |
|---|---|
| New page | `app/<route>/page.tsx` |
| New API endpoint | `app/api/<name>/route.ts` |
| New React component | `components/<feature>/ComponentName.tsx` |
| New reusable UI primitive | `components/ui/Name.tsx` |
| New AI provider | `lib/providers/<name>.ts` + register in `lib/providers/index.ts` |
| New integration | `lib/integrations/<name>.ts` + register call at startup |
| New workflow node type | Type in `lib/workflow/types.ts`, registry entry in `registry.ts`, component in `components/workflow/nodes/` |
| New Zustand store | `lib/store/<name>.ts` |
| New utility function | `lib/utils/<name>.ts` |
| New test | `__tests__/<mirror-of-source-path>.test.ts(x)` |

## Key Patterns

### Provider Pattern (lib/providers/)

All AI providers implement the `AIProvider` interface:

```typescript
interface AIProvider {
  name: ProviderName;
  models: AIModel[];
  chat(messages: ChatMessage[], modelId: string, onChunk: StreamCallback, config: ProviderConfig): Promise<void>;
  testConnection(config: ProviderConfig): Promise<boolean>;
}
```

`ProviderName` is `'openai' | 'anthropic' | 'google' | 'local'`. Each provider (openai.ts, anthropic.ts, gemini.ts, local.ts) implements streaming via its native SDK or the OpenAI-compatible local API. The `chat()` method calls `onChunk(text)` for each token. The server API route uses `createSSEEncoder()` to wrap chunks as SSE events.

**LocalProvider** (`lib/providers/local.ts`) is special:
- Uses the OpenAI SDK with a custom `baseURL` pointing to Ollama (`http://localhost:11434/v1`) or LM Studio (`http://localhost:1234/v1`).
- Model IDs are prefixed: `ollama/<model>` or `lmstudio/<model>`. The prefix determines which server to route to.
- No API key is required. `apiKey: 'not-required'` satisfies the SDK.
- `discoverLocalModels()` is an exported server-side helper used by `GET /api/models?includeLocal=true`.
- On connection failure, the error is translated to a user-friendly "Is Ollama running?" message.

### Personas Pattern (lib/store/personas.ts)

The `usePersonasStore` Zustand store manages AI personas with Zustand `persist` middleware:

- **5 built-in personas** (`builtin-general`, `builtin-code`, `builtin-writer`, `builtin-analyst`, `builtin-onboarding`) are always seeded from the constant `BUILT_IN_PERSONAS` array.
- Custom personas are persisted to `localStorage` under the key `lucy-personas`.
- On rehydrate, built-in personas are re-merged with stored custom ones so built-ins always stay up to date.
- Built-in personas cannot be deleted (`deletePersona` is a no-op for IDs starting with `builtin-`).
- The `activePersonaId` is also persisted. Defaults to `builtin-general`.
- `getActivePersona()` returns the full `Persona` object for the active ID, or `null`.

The `PersonaSelector` component (in `components/chat/PersonaSelector.tsx`) renders as a compact chip in the chat input bar and opens a `role="listbox"` dropdown. Clicking "Create Custom" navigates to `/personas`.

The `/personas` page (`app/personas/page.tsx`) shows all personas as cards with inline editors for creating/editing custom ones.

### Theme System (lib/theme.ts + components/ThemeProvider.tsx)

Five themes — `luminous` (default for new users), `industrial`, `editorial`, `dark` (legacy "Minimal dark"), `light`. Mechanism:

- `<html>` carries `class="dark|light"` (so `dark:` utilities keep working) **plus** `data-theme="luminous|industrial|editorial"` for brand themes.
- Each theme defines CSS variables in `app/globals.css` (`--bg --surface --raised --edge --edge-strong --accent --accent-soft --t1 --t2 --t3 --radius --glow`), mapped in `tailwind.config.ts` to semantic utilities: `bg-bg/surface/raised`, `border-edge/edge-strong`, `text-t1/t2/t3`, `bg-accent(-soft)`, `rounded-theme`, `shadow-glow(-sm)`. **Use these tokens, not gray-* literals, in app chrome.**
- `lib/theme.ts` is the model: `BRAND_THEMES`, `Theme`, `DEFAULT_THEME`, `resolveThemeAttrs()`, `applyThemeToDocument()`, `THEME_OPTIONS` (picker metadata).
- Per-theme flourishes hook on classes `msg-user`, `msg-assistant`, `role-label`, `btn-primary` (CSS in globals.css; e.g. editorial shows uppercase role labels, luminous tints user rows).
- The font is **Manrope** via `next/font/google` (`--font-manrope`, Tailwind `font-sans`).

Flash-of-wrong-theme prevention (two parts):
1. **Inline script in `app/layout.tsx`** — reads `lucy-settings` from localStorage and applies class + `data-theme` synchronously before React hydrates. It duplicates `lib/theme.ts` logic as a string — KEEP THEM IN SYNC.
2. **`<ThemeProvider>` component** — calls `applyThemeToDocument()` whenever the Zustand `theme` changes.

Do not modify `layout.tsx` to hard-code a dark class on `<html>` — the inline script handles initial hydration correctly. The theme picker lives in Settings → General; the Topbar sun/moon button toggles light ↔ luminous.

### Auth System (lib/supabase/auth.tsx + proxy.ts)

`AuthProvider` is a React context that wraps the entire app. It:
- Exposes `user`, `session`, `loading`, `authEnabled`, `signIn`, `signUp`, `signOut`, `signInWithGoogle`, and `resetPassword`.
- In standalone mode (`authEnabled === false`), all auth methods are no-ops and `user` is always null.
- In connected mode, it loads the session on mount and subscribes to `onAuthStateChange` for real-time session updates.

`proxy.ts` runs at the edge on every request:
- Standalone mode: calls `NextResponse.next()` immediately.
- Connected mode: creates a server-side Supabase client, reads the session from cookies, and redirects to `/auth/login` if the session is missing for protected routes (`/chat`, `/workflows`, `/settings`).
- Public paths (`/auth/*`, `/api/*`, `/onboarding`, `/embed`, `/personas`) are never blocked.

### Storage Adapter Pattern (lib/storage/)

The `StorageAdapter` interface defines CRUD operations for conversations, messages, preferences, and provider configs. Two implementations exist:

- `LocalStorageAdapter` -- reads/writes browser localStorage (standalone mode)
- `SupabaseStorageAdapter` -- reads/writes Supabase PostgreSQL (connected mode)

Detection is automatic: if `NEXT_PUBLIC_SUPABASE_URL` is set, Supabase is used. The `StorageProvider` React context exposes the adapter via `useStorage()`. All Zustand store mutations accept `adapter` as a parameter and write through to storage.

### Token Estimation (lib/utils/tokens.ts)

Token estimation uses the simple heuristic of `chars / 4` (OpenAI's rule of thumb for English text). This avoids shipping the heavy `tiktoken` WASM to the browser. Accuracy is within ~10-15%.

- `estimateTokens(text)` -- returns token count for a single string
- `estimateConversationTokens(messages)` -- sums all messages + 4 token overhead per message

Token counts are shown per-message on hover (in `ChatMessage`) and as a conversation total in the `ChatWindow` header.

### Code Block Rendering (components/chat/ChatMessage.tsx)

The `CodeBlock` component renders inside `ReactMarkdown`'s `pre` component override:
- Extracts the language from `className="language-*"` on the `code` child
- Shows a header bar with the language label and a copy-to-clipboard button
- For blocks with more than 5 lines, renders a `<table>` with non-selectable line numbers in the left column
- Horizontally scrollable via `overflow-x-auto`
- Uses `rehype-highlight` for syntax colouring

### Message Edit and Regenerate (components/chat/ChatMessage.tsx)

- **Edit** (user messages only): clicking the pencil icon switches the message to an inline `<textarea>`. Cmd/Ctrl+Enter saves; Escape cancels. Saving calls `onEditSave(index, newContent)` — the parent chat page truncates the conversation at that index and re-sends.
- **Regenerate** (assistant messages only): clicking the regenerate icon calls `onRegenerate(index)` — the parent truncates the conversation at that message and re-sends the preceding user message.

### Mobile Sidebar (components/chat/ChatSidebar.tsx)

When `mobileOverlay={true}`:
- Renders a fixed backdrop (`bg-black/60`, `aria-hidden="true"`) behind the sidebar
- The sidebar panel slides in as a fixed overlay from the left
- A close button (X icon) is shown in the sidebar header
- Touch events track a swipe gesture: a left-swipe of >60px calls `onMobileClose`
- Selecting any conversation calls `onMobileClose` to auto-close the panel

### PWA Icons (app/icon.tsx, app/icon-192.tsx, app/icon-512.tsx)

All three icon files use `next/og` (`ImageResponse`) with `runtime = 'edge'`:
- `app/icon.tsx` -- 32×32, used as the browser favicon
- `app/icon-192.tsx` -- 192×192, referenced in `app/manifest.ts` for PWA home screen
- `app/icon-512.tsx` -- 512×512, referenced in `app/manifest.ts` for PWA splash screen
- All render a `linear-gradient(135deg, #8B5CF6, #6D28D9)` circle with a white "L"

### Integration Registry (lib/integrations/)

External apps describe their data schema and actions via `ProjectIntegration`:

```typescript
interface ProjectIntegration {
  id: string;
  name: string;
  tables: TableDefinition[];     // What Lucy can read
  actions: ActionDefinition[];   // What Lucy can do
  supabaseSchema?: string;       // Schema prefix for queries
}
```

Call `registerProject(integration)` at startup. The context builder (`context.ts`) queries registered tables and injects a summary into the AI system prompt. The action executor (`actions.ts`) handles `supabase-insert`, `supabase-update`, `api-call`, and `workflow` handler types.

### Zustand Stores (lib/store/)

Four stores manage application state:

1. **useChatStore** -- ephemeral UI state: selected model, streaming content, loading/error flags. Not persisted.
2. **useConversationsStore** -- conversation list with messages. Persisted via write-through to StorageAdapter. Messages loaded lazily per conversation.
3. **useSettingsStore** -- API keys, theme, default model, `ollamaUrl`, `lmStudioUrl`. Persisted via write-through to StorageAdapter.
4. **usePersonasStore** -- persona list and active persona ID. Persisted to `localStorage` via Zustand `persist` middleware (not StorageAdapter). Built-in personas are always re-seeded on hydration.

All StorageAdapter-persisted stores are bootstrapped by `<StoreSync>` on mount. Store mutations that persist data accept the adapter as a parameter: `setApiKey(provider, key, adapter)`.

### Streaming (SSE Pattern)

Server to client streaming uses Server-Sent Events:

- **Server** (`lib/utils/stream.ts`): `createSSEEncoder()` produces `data: {"content":"..."}\n\n` events, ending with `data: [DONE]\n\n`
- **Client** (`lib/utils/stream.ts`): `parseSSEStream()` reads the `ReadableStream`, parses SSE lines, calls `onChunk`/`onDone`/`onError` callbacks
- The API route uses `new ReadableStream({ async start(controller) { ... } })` to stream the response

### Workflow Engine (lib/workflow/engine.ts)

The `WorkflowEngine` class executes a workflow graph:

1. Finds the Start node
2. Seeds context from input variables
3. Traverses nodes following edges (topological order)
4. Each node type has a dedicated executor (runLLM, runCondition, runTransform, etc.)
5. LLM nodes call real AI APIs via `lib/providers`
6. Condition nodes set `branch:<nodeId>` in context to control edge filtering
7. Variables are interpolated with `{{varName}}` syntax
8. Execution produces logs with per-node timing, status, and output

## Testing

### Overview

Lucy has **219 tests across 33 test suites** covering lib utilities, storage, workflow engine, provider registry, integration registry, memory system, admin API, and React components.

### Running Tests

```bash
# Run all tests once
npm test

# Run in watch mode (re-runs on file change)
npm run test:watch

# Type-check without running
npx tsc --noEmit
```

### Configuration

- **Config file**: `jest.config.ts` -- uses `nextJest()` wrapper for Next.js compatibility, `jsdom` environment, `@/` module alias pointing to project root
- **Setup file**: `jest.setup.ts` -- imports `@testing-library/jest-dom` for extended matchers
- **Test match**: `**/__tests__/**/*.(test|spec).(ts|tsx)`

### Test Suites

| Suite | Tests cover |
|---|---|
| `__tests__/lib/providers/index.test.ts` | `getProvider`, `getProviderForModel`, `getAllModels`, `getModelsByProvider` — all 4 providers including local |
| `__tests__/lib/storage/local.test.ts` | `LocalStorageAdapter` CRUD: conversations, messages, preferences, provider configs |
| `__tests__/lib/workflow/engine.test.ts` | Start→Output execution, edge following, condition branching (true/false), transform operations |
| `__tests__/lib/integrations/registry.test.ts` | `registerProject`, `getProject`, `getAllProjects`, `getProjectTables`, `getProjectActions` |
| `__tests__/lib/utils/markdown.test.ts` | `hasMarkdown`, `generateConversationTitle`, `truncate` |
| `__tests__/components/chat/ModelSelector.test.tsx` | Renders provider groups (OpenAI/Anthropic/Google), `onChange` fires with correct model id |
| `__tests__/components/ui/Button.test.tsx` | Renders children, variant classes, loading spinner, disabled state, click behaviour |

### Patterns for New Tests

- Place test files under `__tests__/` mirroring the source path (e.g., `lib/utils/tokens.ts` → `__tests__/lib/utils/tokens.test.ts`)
- Mock heavy SDK dependencies at the top of the file using `jest.mock()`
- Use an in-memory localStorage mock for tests involving `LocalStorageAdapter`
- Test React components with `render()` from `@testing-library/react` and `screen` queries
- For components that fetch (like ModelSelector), mock `global.fetch` to return a controlled response
- Avoid testing implementation details; test observable behaviour and outputs
- All provider tests should mock the SDK classes so no real API keys are needed

## Development Guide

### Run Locally

```bash
# Install dependencies
npm install

# Copy env template and fill in your API keys
cp .env.example .env.local

# Start dev server
npm run dev
# App runs at http://localhost:3000
```

### Docker

```bash
# Build and run with docker-compose
docker-compose up --build

# Build image manually
docker build -t lucy-ai .

# Run manually
docker run -p 3000:3000 -e OPENAI_API_KEY=sk-... lucy-ai
```

The `docker-compose.yml` maps `OLLAMA_URL` and `LM_STUDIO_URL` to `host.docker.internal` so the containerised app can reach Ollama/LM Studio running on the host machine.

### Environment Variables

| Variable | Required | Description |
|---|---|---|
| `OPENAI_API_KEY` | No | Server-side fallback; users can also set keys in Settings |
| `ANTHROPIC_API_KEY` | No | Server-side fallback for Anthropic |
| `GOOGLE_API_KEY` | No | Server-side fallback for Google AI |
| `DEEPSEEK_API_KEY` | No | Server-side fallback for DeepSeek |
| `GROQ_API_KEY` | No | Server-side fallback for Groq |
| `MISTRAL_API_KEY` | No | Server-side fallback for Mistral |
| `XAI_API_KEY` | No | Server-side fallback for xAI (Grok) |
| `OPENROUTER_API_KEY` | No | Server-side fallback for OpenRouter |
| `NEXT_PUBLIC_SUPABASE_URL` | No | Enables Supabase mode + auth when set |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | No | Required alongside SUPABASE_URL |
| `OLLAMA_URL` | No | Ollama server URL; default `http://localhost:11434` |
| `LM_STUDIO_URL` | No | LM Studio server URL; default `http://localhost:1234` |

Without any env vars, Lucy runs in standalone mode using localStorage. Users configure API keys through the Settings page. Local models need no env vars -- Lucy probes localhost automatically.

### How to Add a New AI Provider

1. Create `lib/providers/<name>.ts` implementing the `AIProvider` interface
2. Add the provider name to the `ProviderName` union type in `lib/providers/types.ts`
3. Add model definitions to `ALL_MODELS` in `lib/providers/types.ts`
4. Register the provider in `lib/providers/index.ts` in the `providers` record
5. Add the API key header mapping in `app/api/chat/route.ts` (`headerMap`)
6. Add the provider to the Settings page `PROVIDERS` array in `app/settings/page.tsx`
7. Add env var fallback in `app/api/chat/route.ts`
8. Add a label entry in `PROVIDER_LABELS` in `components/chat/ChatMessage.tsx` and `ModelSelector.tsx`

### How to Work With Auth

**Add a protected page**: The middleware already protects `/chat`, `/workflows`, and `/settings`. To protect a new route, add its prefix to `protectedPrefixes` in `proxy.ts`.

**Access the current user in a component**:
```typescript
import { useAuth } from '@/lib/supabase/auth';
const { user, authEnabled } = useAuth();
```

**Access the current user in an API route**:
```typescript
import { createServerClient } from '@supabase/ssr';
// See lib/memory/auth.ts (resolveMemoryAuth) for the cookie-based server client pattern
```

**Add a new auth provider (e.g., GitHub)**: Enable it in your Supabase project dashboard, then add a button in `app/auth/login/page.tsx` that calls `supabase.auth.signInWithOAuth({ provider: 'github', ... })`.

### How to Add a Persona

**To add a new built-in persona**: Add an entry to the `BUILT_IN_PERSONAS` array in `lib/store/personas.ts`. The ID must start with `builtin-`. The persona will appear for all users and cannot be deleted.

**To create a custom persona at runtime**: Use the `/personas` page UI, or call `usePersonasStore.getState().addPersona(data)` programmatically. Custom personas are persisted to `localStorage`.

### How to Add a New Workflow Node Type

1. Add the type name to `NodeType` union in `lib/workflow/types.ts`
2. Define a `<Name>NodeConfig` interface extending `NodeConfigBase`
3. Add the config to `NodeConfig` union type
4. Add a default config to `NODE_CONFIG_DEFAULTS`
5. Add a `NodeTypeDefinition` entry to `NODE_TYPE_REGISTRY` in `lib/workflow/registry.ts`
6. Create `components/workflow/nodes/<Name>Node.tsx` extending `BaseNode`
7. Register the component in `NODE_TYPES` in `WorkflowCanvas.tsx`
8. Add executor method `run<Name>()` in `lib/workflow/engine.ts`
9. Add the case to the `runNode()` switch
10. Add config UI to `NodeConfigPanel.tsx`

### How to Add a New Integration

1. Create `lib/integrations/<name>.ts` following the pattern of `contractors-room.ts`
2. Define the `ProjectIntegration` object with tables and actions
3. Export a `register<Name>()` function that calls `registerProject()`
4. Call the register function at app startup:
   - Server side: in `app/api/chat/route.ts` (top-level)
   - Client side: in `app/settings/integrations/page.tsx`
5. If the integration has Supabase tables, use the `supabaseSchema` field

### How to Add a New Page

1. Create `app/<route>/page.tsx`
2. Use `'use client'` directive if the page has interactivity
3. Wrap the page with `<AppShell>` for authenticated app pages (not `/auth/*` or `/embed`)
4. Follow the existing layout pattern: full-height flex column, scrollable main area
5. If the page should be protected by auth, add its prefix to `protectedPrefixes` in `proxy.ts`
6. If the page needs a loading skeleton, add `app/<route>/loading.tsx`

### How to Run Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run a specific test file
npx jest __tests__/lib/utils/markdown.test.ts

# Run tests matching a pattern
npx jest --testNamePattern="getProvider"
```

When writing new tests:
- Mock all external SDK dependencies at the top of the file with `jest.mock()`
- Use the `jsdom` environment (default) for component tests
- Use `@testing-library/react` `render` + `screen` + `fireEvent` for React components
- Mirror the source file path under `__tests__/`

## Database

### Schema Overview (lib/supabase/schema.sql)

All tables live in the `lucy` schema (NOT `public`). The schema provides namespacing, so tables have clean names (no prefix). Supabase clients must be configured with `db: { schema: 'lucy' }` and `lucy` must be in `PGRST_DB_SCHEMAS`.

| Table | Purpose |
|---|---|
| `conversations` | Chat conversation metadata (user, title, model, provider) |
| `messages` | Individual chat messages (role, content, tokens) |
| `provider_configs` | Encrypted API keys per user per provider |
| `user_preferences` | Theme, default model, company name |
| `workflows` | Workflow definitions (nodes/edges stored as JSONB) |
| `workflow_runs` | Execution history (inputs, outputs, logs, status) |
| `screenings` | AI screening records (grade, questions, transcript) |
| `screening_answers` | Individual Q&A answers per screening |
| `api_keys` | Per-user API keys (SHA-256 hashed, prefix for display) |

### Table Relationships

```
conversations (user_id -> auth.users)
  +-- messages (conversation_id -> conversations)

provider_configs (user_id -> auth.users, unique per user+provider)
user_preferences (user_id -> auth.users, 1:1)

workflows (user_id -> auth.users)
  +-- workflow_runs (workflow_id -> workflows, user_id -> auth.users)

screenings (created_by -> auth.users)
  +-- screening_answers (screening_id -> screenings)

api_keys (user_id -> auth.users)
```

### RLS Policy Design

Every table has RLS enabled. Core pattern: `auth.uid() = user_id`. Screenings use `created_by = auth.uid()` (only the API key owner sees their screenings). Service-role (used by API routes) bypasses RLS.

### Migration Strategy

- All Lucy tables live in the `lucy` schema
- Schema changes must be reflected in `lib/supabase/schema.sql`
- Run SQL files via: `docker exec -i supabase-db psql -U supabase_admin -d postgres < file.sql`
- Always add `IF NOT EXISTS` to CREATE statements
- DDL must use `supabase_admin` user (not `postgres`)

## API Routes

### POST /api/chat

Streams an AI response via SSE. Rate limited to 30 requests per IP per minute.

**Request headers**:
- Cloud providers: `x-openai-key`, `x-anthropic-key`, or `x-google-key` (optional, falls back to env vars)
- Local provider: `x-ollama-url`, `x-lmstudio-url` (optional, fall back to env vars and then to localhost defaults)

**Request body**:
```json
{
  "messages": [{ "role": "user", "content": "Hello" }],
  "model": "gpt-4o",
  "provider": "openai",
  "projectId": "contractors-room",  // optional
  "userId": "uuid"                   // optional
}
```

**Response**: `text/event-stream`
```
data: {"content":"Hello"}

data: {"content":" there!"}

data: [DONE]
```

**Rate limit response** (HTTP 429):
```json
{ "error": "Rate limit exceeded. Please wait 42 seconds before trying again." }
```

### GET /api/models

Returns all available models grouped by provider.

`?includeLocal=true` additionally probes Ollama (`OLLAMA_URL` / default localhost:11434) and LM Studio (`LM_STUDIO_URL` / default localhost:1234) for currently-loaded models, and returns a `localStatus` object.

**Response** (without `?includeLocal=true`):
```json
{
  "models": [{ "id": "gpt-4o", "name": "GPT-4o", "provider": "openai", ... }],
  "byProvider": { "openai": [...], "anthropic": [...], "google": [...], "local": [...] }
}
```

**Response** (with `?includeLocal=true`):
```json
{
  "models": [...],
  "byProvider": { ..., "local": [{ "id": "ollama/llama3.1", ... }] },
  "localStatus": {
    "ollama": { "available": true, "url": "http://localhost:11434", "modelCount": 2 },
    "lmstudio": { "available": false, "url": "http://localhost:1234", "modelCount": 0 }
  }
}
```

### GET /api/embed

Returns a JavaScript snippet for embedding Lucy as a widget. Query params: `project`, `model`, `position`, `theme`.

### GET /auth/callback

Supabase OAuth redirect handler. Exchanges the auth code for a session cookie and redirects to `/chat` (or the original `redirectTo` path).

## Common Tasks

**I want to change the default model:**
Set `selectedModel` default in `lib/store/chat.ts` and `defaultModel` in `lib/store/settings.ts`.

**I want to add a new UI primitive:**
Create `components/ui/Name.tsx`, export the component. Follow the pattern in `Button.tsx` or `Card.tsx`.

**I want to modify the chat system prompt:**
Edit the system message construction in `app/api/chat/route.ts` (around the `messagesWithContext` block).

**I want to add a new Supabase table:**
Add the CREATE TABLE statement to `lib/supabase/schema.sql` with the `lucy_` prefix. Add RLS policy. Add index. Update the `SupabaseStorageAdapter` if the table is accessed through the storage interface.

**I want to add a model to an existing provider:**
Add the model definition to `ALL_MODELS` in `lib/providers/types.ts`. That is the only change needed.

**I want to persist a new piece of data:**
Add the field to the `StorageAdapter` interface in `lib/storage/index.ts`. Implement in both `local.ts` and `supabase.ts`. Add the Supabase column to `schema.sql`.

**I want to change the brand color:**
Edit the `lucy` color scale in `tailwind.config.ts` AND the per-theme `--accent`/`--accent-soft` token values in `app/globals.css`.

**I want to tweak a theme's look:**
Edit that theme's CSS variable block in `app/globals.css` (`[data-theme='…']` or `.dark`/`:root`). Add per-theme flourishes as `[data-theme='…'] .hook-class` rules.

**I want to embed Lucy in another app:**
Use the embed script: `<script src="https://your-lucy/api/embed?project=your-project" async></script>`. Or import `LucyWidget` directly in a React app.

**I want to protect a new route with auth:**
Add the route prefix to `protectedPrefixes` in `proxy.ts`.

**I want to add a new auth provider (e.g., GitHub):**
Enable it in Supabase project settings → Authentication → Providers. Add a button in `app/auth/login/page.tsx` calling `signInWithOAuth({ provider: 'github' })`.

**I want to add a new local model server (e.g., Jan.ai):**
Add a new URL env var (e.g., `JAN_URL`). Add a new model prefix (e.g., `jan/`). Extend `resolveBaseUrl()` and `discoverLocalModels()` in `lib/providers/local.ts`. Add a URL input in the Local Models section of `app/settings/page.tsx`.

**I want to add a new built-in persona:**
Add an entry to `BUILT_IN_PERSONAS` in `lib/store/personas.ts`. The `id` must start with `builtin-`. The persona will be available to all users immediately after the next deployment.

**I want to run the tests:**
Run `npm test` for a single pass or `npm run test:watch` for watch mode. New test files go under `__tests__/` mirroring the source path.

**I want to build and run with Docker:**
Run `docker-compose up --build` from the project root. Set env vars in your shell or a `.env` file before running.

## Known Issues / Technical Debt

1. **Workflow storage is localStorage-only**: `LocalWorkflowStorage` always uses localStorage even when Supabase is enabled. The `lucy_workflows` table exists in schema.sql but there is no `SupabaseWorkflowStorage` implementation.

2. **Knowledge Base node is a placeholder**: The `runKnowledgeBase()` method in the workflow engine returns a hardcoded placeholder string. No vector store is connected.

3. **Workflow action handler is a placeholder**: The `handleWorkflow` function in `actions.ts` makes a fetch to `/api/workflows/[id]/run` which does not exist as an API route.

4. **eslint-disable comments**: Several `useEffect` hooks have `// eslint-disable-next-line react-hooks/exhaustive-deps` to suppress missing dependency warnings. These should be reviewed to ensure they are intentional.

5. **No pagination for conversations**: All conversations are loaded at once from storage. This will degrade performance as conversation count grows.

6. **API keys in localStorage are not encrypted**: In standalone mode, API keys are stored as plain text in localStorage (local to the user's own browser). In connected mode, keys are AES-256-GCM encrypted server-side via `POST /api/provider-keys` (`lib/auth/provider-keys.ts`); legacy XOR rows are migrated on read. See `docs/security/2026-06-10-security-hardening.md`.

7. **Local model URLs are in-memory only**: `ollamaUrl` and `lmStudioUrl` in the settings store are not yet persisted to the StorageAdapter. They reset to defaults on page reload.

8. **Rate limiter resets on server restart**: The IP rate limiter in `app/api/chat/route.ts` uses an in-memory `Map`. In serverless deployments each function invocation has its own memory, so the limiter provides limited protection. Use Redis or Upstash for a production-grade rate limiter.

9. **Personas not in Supabase**: The `usePersonasStore` persists custom personas to browser `localStorage` via Zustand persist middleware, not to the `StorageAdapter`. In Supabase (connected) mode, custom personas are not synced across devices.

10. **No test for token utilities**: `lib/utils/tokens.ts` (estimateTokens, estimateConversationTokens) is not yet covered by a test suite.

## Do NOT

- **Do not modify `lib/supabase/schema.sql` without updating both storage adapters** (`lib/storage/local.ts` and `lib/storage/supabase.ts`) to match.
- **Do not store API keys in plain text in Supabase**. Provider keys must go through `POST /api/provider-keys`, which AES-256-GCM encrypts them server-side (`lib/auth/provider-keys.ts`). Never write `provider_configs.api_key_encrypted` directly from the client.
- **Do not add `'use client'` to files in `lib/` that are imported by API routes**. API routes run on the server. If a lib file needs to work in both contexts, do not add the directive.
- **Do not define `NODE_TYPES` inside a React component's render function**. React Flow requires a stable reference. It must be defined outside the component (see `WorkflowCanvas.tsx`).
- **Do not import from `@xyflow/react` in server components**. React Flow is client-only.
- **Do not remove the `lucy_` prefix from Supabase table names**. It prevents collisions with other apps sharing the database.
- **Do not commit `.env.local` or any file containing real API keys**. The `.gitignore` already excludes `.env*.local`.
- **Do not use `any` type**. Use `unknown` and narrow with type guards. Existing `as` casts in the workflow engine are acceptable since they are behind node-type switches.
- **Do not break the `StorageAdapter` interface contract**. Both implementations must stay in sync. If you add a method, implement it in both `local.ts` and `supabase.ts`.
- **Do not add direct DOM manipulation** in React components. Use refs and React APIs.
- **Do not install a CSS-in-JS library**. The project uses Tailwind exclusively.
- **Do not hard-code `class="dark"` on `<html>` in layout.tsx**. The inline script + ThemeProvider handle the no-flash theme setup correctly.
- **Do not add `'local'` to the `ApiKeys` interface** in `lib/store/settings.ts`. Local models do not need API keys. The `ApiKeys` type covers cloud provider keys only.
- **Do not delete built-in personas by modifying `BUILT_IN_PERSONAS`** without updating any code that references them by ID (e.g., default `activePersonaId` values, tests). Built-in IDs must always start with `builtin-`.
- **Do not skip `jest.mock()` for SDK dependencies in tests**. Tests must not make real network calls or require real API keys. Always mock the provider SDK classes and any external HTTP calls.
- **Do not use the `public` schema for Lucy tables**. All tables must be in the `lucy` schema.
- **Do not create Supabase clients without `db: { schema: 'lucy' }`** — queries will silently hit the wrong schema.

## Screening System

Lucy provides AI-powered contractor screening for the Contractors Room marketplace.

### Two Screening Modes

1. **Profile Verification** (`profile_verification`) — automatic one-shot review of a contractor's profile. Lucy generates a grade (1–5) with strengths and concerns. No human interaction needed.
2. **Project Screening** (`project_screening`) — client-initiated screening. Lucy generates 5–8 tailored questions, the contractor answers, then Lucy grades the fit. Results visible only to the client.

### Grading Scale

| Grade | Label | Meaning |
|---|---|---|
| 5 | Excellent Match | Strong fit on all criteria |
| 4 | Good Fit | Solid candidate with minor gaps |
| 3 | Potential Fit | Some alignment but notable concerns |
| 2 | Weak Fit | Significant mismatches |
| 1 | Not Recommended | Does not meet requirements |

### Screening Pipeline

```
CR proxy → POST /api/screening/start (API key auth)
  → startScreening() creates DB record
  → Profile verification: callLLM() → grade immediately → completed
  → Project screening: callLLM() → generate questions → awaiting_answers
    → POST /api/screening/:id (submit answers)
      → callLLM() → grade → completed
```

### Status Flow

`pending` → `generating_questions` → `awaiting_answers` → `grading` → `completed`
             (profile verification skips questions: `pending` → `grading` → `completed`)
             On error: any state → `failed`

## API Key System

External applications authenticate to Lucy's screening API using per-user API keys.

### Key Format

Keys are `lucy_k_<24-random-base64url-chars>` (e.g. `lucy_k_8AgXcZBucOrB...`). Only the SHA-256 hash is stored in the database. The full key is shown once on creation and never again.

### Authentication Flow

```
External app sends:  Authorization: Bearer lucy_k_...
  → validateApiKey() hashes the key
  → looks up hash in lucy.api_keys
  → returns user_id if active, null otherwise
  → updates last_used_at (fire and forget)
```

### API Key Management Routes

- `POST /api/keys` — create a new key (requires Supabase session token)
- `GET /api/keys` — list keys for the current user (prefix only)
- `DELETE /api/keys?id=<uuid>&action=revoke` — deactivate a key
- `DELETE /api/keys?id=<uuid>&action=delete` — permanently remove a key

### Settings Page

The Settings page has an "API Keys" section (visible only in Supabase mode) where users can create, view, and revoke keys.

### Contractors Room Integration

CR proxies screening requests through `/api/lucy/screen/*` with the `LUCY_API_KEY` env var. The current key is for `admin@contractorsroom.com`.

## MCP Server

Lucy exposes screening tools via the Model Context Protocol (MCP) for use with Claude Code, Cursor, and other MCP-compatible editors.

### Available Tools

| Tool | Description |
|---|---|
| `start_screening` | Start a new contractor screening |
| `get_screening` | Get screening status and results |
| `list_screenings` | List screenings with filters |
| `submit_screening_answers` | Submit contractor answers |
| `verify_contractor_profile` | One-shot profile verification |

### Running the MCP Server

```bash
cd C:\RepositoryAI\LucyAI
npm run mcp
```

### Configuration (claude_desktop_config.json)

```json
{
  "mcpServers": {
    "lucy": {
      "command": "npx",
      "args": ["tsx", "C:\\RepositoryAI\\LucyAI\\lib\\mcp\\server.ts"],
      "env": {
        "NEXT_PUBLIC_SUPABASE_URL": "http://localhost:8000",
        "SUPABASE_SERVICE_ROLE_KEY": "...",
        "LUCY_API_KEY": "lucy_k_..."
      }
    }
  }
}
```

## Memory System

Lucy's durable cross-conversation memory. Design spec: `docs/superpowers/specs/2026-06-08-lucy-memory-system-design.md`. Implementation plan: `docs/superpowers/plans/2026-06-08-lucy-memory-system.md`.

**Module:** `lib/memory/`
- `types.ts` — shared types + extraction Zod schemas (`MemoryType` = semantic | pragmatic | episodic; `MemoryVisibility` = private | project | global; `MemorySource` = extracted | user_remember | user_global | admin).
- `store.ts` — the `MemoryStore` interface. Two backends behind it:
  - `supabase-store.ts` — connected mode: pgvector (HNSW + `halfvec`) vector search **and** Postgres FTS, fused with **Reciprocal Rank Fusion** (`scoring.ts`).
  - `local-store.ts` + `indexeddb-kv.ts` — standalone mode: lexical search over IndexedDB (no embeddings).
- `embeddings.ts` — admin-configurable embedder (OpenAI `text-embedding-3-small` default); returns `null` to degrade to lexical.
- `extractor.ts` — reconciliation-aware end-of-conversation pass (ADD/UPDATE/MERGE/SKIP), Zod-validated, with the `privacy.ts` secret/PII guard.
- `scoring.ts` — source weights, importance, RRF, category-specific decay half-lives, rank score.
- `profile.ts` — field-level profile merge + entity-name normalization.
- `injector.ts` — formats the profile (“Who you are”) + retrieved memories (“What Lucy knows”) into the system prompt.
- `commands.ts` — parses `/remember` and `/global`.
- `server.ts` — `buildRetrievalBlock()` used by the chat route.
- `index.ts` — `createMemoryStore()` factory + `ingestExtraction()` / `ingestCommand()`.

**Data flow:**
- **Capture:** `app/chat/page.tsx` fires `POST /api/memory/extract` (fire-and-forget) when a turn completes. Slash-commands are intercepted client-side: `/remember`+`/global` → `POST /api/memory/command`, `/forget` → `POST /api/memory/forget`, `/memories` → `GET /api/memory/list`; `/incognito`/`/new`/`/help` handled in-page. Incognito skips capture.
- **Retrieval:** `POST /api/chat` injects `buildRetrievalBlock()` into the system prompt when the request carries `x-memory-enabled: 1` and a `userId`.
- **Admin:** `GET/POST /api/memory/settings` (single-row `lucy.memory_settings`); `GET/DELETE /api/memory/list` for the management/usage UI (`components/settings/MemoryPanel.tsx`).

**Gating:** memory is **off by default**. Connected mode: the admin toggle in Settings → Memory writes `memory_settings.enabled`; `useMemoryStore.memoryHeader()` sets `x-memory-enabled`. Standalone mode: a persisted `localEnabled` flag (`useMemoryStore.localActive()`) gates client-side retrieval/capture (IndexedDB) — the server can't reach the browser store, so local retrieval injects the block before sending and capture goes through the stateless `/api/memory/extract-local` endpoint.

**Auth (security):** memory routes derive `userId` **server-side** via `resolveMemoryAuth` (`lib/memory/auth.ts`) — from the Supabase cookie session (RLS-scoped client) or a validated Lucy API key — and **never** trust a body/query `userId`. Settings writes require the admin role (`lucy_role` in auth `app_metadata`, see `lib/auth/admin.ts`). `/api/memory/extract-local` is rate-limited (`lib/api/rate-limit.ts`).

**Chat commands:** the slash-command registry + autocomplete lives in `lib/chat/slash-commands.ts` (`getCommandSuggestions`, `parseSlashCommand`); the menu renders in `components/chat/ChatInput.tsx` and dispatch happens in `app/chat/page.tsx` `handleSend`. Set: `/remember /forget /global /memories /incognito /new /help`.

**Embedder (admin-editable):** Settings → Memory exposes model + base URL + dimensions (with OpenAI / Ollama presets). Changing the dimension reshapes the pgvector column via the `lucy.set_embedding_dim()` RPC (`memory_search.sql`, service-role only). `embeddings.ts` calls an OpenAI-compatible endpoint, so a fully-local setup is `embedder_base_url=http://localhost:11434/v1` + an Ollama embedding model (e.g. `embeddinggemma`, 768-dim) — no API key. NOTE: `gemma3:4b`/`llama3.2` are chat models and cannot embed; use a dedicated embedding model.

**Schema:** `lib/supabase/memory.sql` (tables, HNSW/FTS indexes, RLS, `memory_settings`) + `lib/supabase/memory_search.sql` (hybrid-search RPCs + `set_embedding_dim`). Apply both after `schema.sql`. Self-hosted note: the `postgres` role can't `CREATE` in the `lucy` schema — apply as `supabase_admin`.

**Phasing:** Phase 1 (this) = the engine, wired for **both** connected (Supabase/pgvector) and standalone (IndexedDB) modes. Deferred: L3 knowledge base + “Lucy Documents” + API/MCP exposure (1.5); association graph (`memory_entities` substrate exists but is intentionally unpopulated until then) + dreaming + prediction (2); org tier on a separate Supabase project (C).

## Auth & Security

Custom auth hardening layer on top of Supabase Auth. **Spec:** `docs/superpowers/specs/2026-06-09-auth-security-profile-design.md`. **Plan:** `docs/superpowers/plans/2026-06-09-auth-security-profile.md`. **Branch:** `feat/auth-security-profile`.

### Lucy Tables (applied via `lib/supabase/auth_security.sql` as `supabase_admin`)

| Table | Purpose |
|---|---|
| `lucy.email_verification_codes` | Scrypt-hashed 6-digit codes for password reset and email-OTP 2FA. RLS disabled — service-role only. |
| `lucy.member_devices` | Per-user device fingerprints with browser/OS/IP. RLS: users can select/delete their own rows; inserts/updates via service client. |
| `lucy.user_profiles` | display_name, avatar_url, company, two_factor_email_enabled. RLS: users select/insert/update their own row. |

### SMTP Environment Variables (`.env.local`, never committed)

| Variable | Value / Description |
|---|---|
| `SMTP_HOST` | `smtp.zoho.eu` |
| `SMTP_PORT` | `587` |
| `SMTP_SECURE` | `tls` (STARTTLS on 587; `ssl` = port 465) |
| `SMTP_USER` | `contact@brand.contractors` |
| `SMTP_PASS` | Zoho app password (from shared secrets) |
| `SMTP_FROM_NAME` | `Lucy` |
| `SMTP_FROM_EMAIL` | `contact@brand.contractors` |

### Email Library (`lib/email/`)

- `smtp.ts` — `loadSmtpConfig()` reads the `SMTP_*` env vars; `getTransport()` returns a cached nodemailer transporter (STARTTLS-safe: `secure:false` on port 587). Returns `null` when SMTP is unconfigured so callers degrade gracefully.
- `templates.ts` — `renderEmail(key, vars)` produces `{ subject, html, text }` for `'passwordReset'` and `'twoFactorCode'` templates. Lucy-branded purple header.
- `codes.ts` — `hashCode` / `checkCode` (Node `crypto` scrypt, per-code salt); pure `evaluateCode(row, code, nowMs)` returning a `Verdict` (`ok | no_code | expired | too_many | mismatch`); `createCode` / `confirmCode` for DB interaction. `CODE_TTL_MINUTES = 15`, `MAX_ATTEMPTS = 5`. Fully unit-tested in `__tests__/lib/email/codes.test.ts`.
- `send.ts` — `sendTemplateEmail(to, key, vars)` — non-throwing wrapper; logs and returns `false` on SMTP failure.

### Password Recovery Flow

1. User submits email on `/auth/forgot-password` → `POST /api/auth/reset/request` — looks up user by email via `auth.admin.listUsers()`, creates a code row, sends a `passwordReset` email. Response is always `{ ok: true }` (no enumeration).
2. User is redirected to `/auth/reset-password?email=…` — enters the 6-digit code + new password → `POST /api/auth/reset/confirm` — verifies code, calls `auth.admin.updateUserById` to set the password.

### Two-Factor Authentication Design

**TOTP (authenticator app):**
- Enroll: `/auth/two-factor-setup` calls `sb.auth.mfa.enroll({ factorType: 'totp' })`, shows QR + manual secret, verifies with `mfa.verify`.
- Challenge at login: `/auth/two-factor-challenge` calls `mfa.listFactors()` → `mfa.challenge()` → `mfa.verify()`. Five failures → sign out → `/auth/account-locked`.

**Email-OTP:**
- Toggle in Settings → Security writes `user_profiles.two_factor_email_enabled`.
- Challenge at login: `/auth/2fa` fires `POST /api/auth/2fa/request` on mount (sends code to user's email), then submits code via `POST /api/auth/2fa/verify`.

**Session gate — `lib/auth/twofa-session.ts`:**
- `set2faPassed(userId)` / `is2faPassed(userId)` / `clear2faPassed()` — per-tab `sessionStorage` key `lucy-2fa-passed`.
- On successful login, `app/auth/login/page.tsx` checks `mfa.listFactors()` and `user_profiles.two_factor_email_enabled`; routes to the appropriate challenge page (TOTP first, then email-OTP) or directly to `/chat`.
- `signOut` calls `clear2faPassed()`.

### Device Tracking

- **Client:** `lib/auth/device.ts` — `trackDevice()` computes a lightweight browser fingerprint (UA + language + screen + timezone, djb2-hashed), calls `POST /api/auth/devices/track`. Called fire-and-forget from `lib/supabase/auth.tsx` on `SIGNED_IN` (once per session, not on token refresh).
- **Routes:** `POST /api/auth/devices/track` — service-client upsert on `(user_id, fingerprint)`, marks previous devices `is_current: false`. `GET /api/auth/devices` — cookie-client list (RLS-scoped). `DELETE /api/auth/devices?id=` — service-client delete scoped to owner.
- **UI:** Settings → Security "Devices & sessions" card — live list with browser/OS/IP/last-active, "This device" badge, Remove button.

### Profile Page (`app/account/profile/page.tsx`)

Backed by `lucy.user_profiles` via `getSupabaseClient()` (browser client, `db: { schema: 'lucy' }`). Fields: email (read-only from `auth.getUser()`), display_name, avatar_url, company. Load: `select('*').eq('user_id', uid).maybeSingle()`. Save: `upsert({ user_id, display_name, avatar_url, company, updated_at })` with `onConflict: 'user_id'` on Save button click.

### Security Page (`app/account/security/page.tsx`)

Three cards:
1. **Change password** — `sb.auth.updateUser({ password })`.
2. **Two-factor authentication** — TOTP status via `mfa.listFactors()` (Enable link → `/auth/two-factor-setup` or Disable via `mfa.unenroll`); Email-2FA toggle bound to `user_profiles.two_factor_email_enabled`.
3. **Devices & sessions** — live list from `GET /api/auth/devices`; removable.

## Deployment Modes

### Standalone (Local)

No Supabase needed. All data in browser localStorage. No auth, no multi-tenancy. Good for personal use or evaluation.

### SaaS (Supabase)

Connected to Supabase PostgreSQL. Full auth, multi-tenancy, RLS. Each user's conversations, workflows, screenings, and API keys are isolated. Required for the Contractors Room integration.

### Shared Supabase with Contractors Room

Both apps share the same local Docker Supabase instance:
- Lucy uses the `lucy` schema
- Contractors Room uses the `contractors_room` schema
- PostgREST exposes both: `PGRST_DB_SCHEMAS=public,storage,graphql_public,cj,contractors_room,lucy`

## Environment Variables (Complete)

| Variable | Required | Description |
|---|---|---|
| `OPENAI_API_KEY` | No | Server-side fallback for OpenAI |
| `ANTHROPIC_API_KEY` | No | Server-side fallback for Anthropic |
| `GOOGLE_API_KEY` | No | Server-side fallback for Google AI |
| `DEEPSEEK_API_KEY` | No | Server-side fallback for DeepSeek |
| `GROQ_API_KEY` | No | Server-side fallback for Groq |
| `MISTRAL_API_KEY` | No | Server-side fallback for Mistral |
| `XAI_API_KEY` | No | Server-side fallback for xAI (Grok) |
| `OPENROUTER_API_KEY` | No | Server-side fallback for OpenRouter |
| `NEXT_PUBLIC_SUPABASE_URL` | No | Enables Supabase mode + auth when set |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | No | Required alongside SUPABASE_URL |
| `SUPABASE_SERVICE_ROLE_KEY` | No | Required for screening API + connected memory (bypasses RLS) |
| `LUCY_ADMIN_EMAIL` | No | Legacy bootstrap only: picks which account is auto-promoted to the first admin. Roles live in auth `app_metadata` (`lucy_role`) and are managed in /admin |
| `LUCY_URL` | No | Lucy's own URL (for MCP callbacks) |
| `LUCY_API_KEY` | No | API key for MCP server and testing |
| `OLLAMA_URL` | No | Ollama server URL; default `http://localhost:11434` |
| `LM_STUDIO_URL` | No | LM Studio server URL; default `http://localhost:1234` |

## Connectors / MCP Marketplace

Lucy provides a per-user MCP connector marketplace: browse a curated catalog, install and configure connectors (with encrypted secrets), and have Lucy's AI call those connectors' tools during chat. **Spec:** `docs/superpowers/specs/2026-06-09-connectors-mcp-marketplace-design.md`. **Plan:** `docs/superpowers/plans/2026-06-09-connectors-mcp-marketplace.md`. **Branch:** `feat/connectors-marketplace`.

### Database Tables (apply `lib/supabase/mcp.sql` as `supabase_admin`)

| Table | Purpose |
|---|---|
| `lucy.mcp_servers` | Curated catalog of connectors (slug, name, category, transport, config_schema, tools). Public read via RLS; writes service-role only. |
| `lucy.mcp_installations` | Per-user connector installs (user_id, server_slug, encrypted config, enabled, require_approval). RLS: each user owns their rows. |

Both tables live in the `lucy` schema. Apply the SQL file once: `docker exec -i supabase-db psql -U supabase_admin -d postgres < lib/supabase/mcp.sql`.

### Catalog Seed

`lib/mcp/catalog.ts` — `CATALOG` array (10 connectors: GitHub, Slack, Notion, Postgres, Linear, Stripe, Brave Search, Filesystem, Fetch, Contractors Room built-in) + `seedCatalog(svcClient)` which idempotently upserts the catalog via `ON CONFLICT (slug) DO UPDATE`. The registry route seeds lazily on first GET when the table is empty.

### `lib/mcp/` Modules

| File | Purpose |
|---|---|
| `types.ts` | `Transport`, `Category`, `ConfigField`, `ToolInfo`, `CatalogServer`, `Installation` |
| `catalog.ts` | `CATALOG` constant (10 connectors) + `seedCatalog()` |
| `registry.ts` | `listCatalog(svc, {category?, q?})` + `getServer(svc, slug)` — read from `mcp_servers` |
| `installer.ts` | `validateConfig`, `maskConfig`, `encodeConfig`, `decodeConfig`, `install`, `uninstall`, `patchInstall`, `getInstallations` |
| `secret.ts` | AES-256-GCM encrypt/decrypt (`encryptSecret`, `decryptSecret`). Key derived from `SUPABASE_SERVICE_ROLE_KEY` via `scrypt`. Ciphertext format: `iv:tag:ct` (hex). Server-only. |
| `client.ts` | `connect(server, config) → McpConn` — short-lived MCP client; stdio (npx launch) or HTTP/SSE transport via `@modelcontextprotocol/sdk`. Returns `{ listTools, callTool, close }`. |
| `loader.ts` | `loadToolsForUser(svc, userId) → LoadedTool[]` — connects to each enabled install, lists its tools (namespaced), closes connections. Built-in connectors are skipped (handled via the integration path). |
| `tool-format.ts` | `NAMESPACE_SEP = '__'`. `toOpenAITools`, `toAnthropicTools`, `qualified`, `parseQualified` — converts `LoadedTool[]` to provider-native tool definitions with `slug__tool_name` namespacing. |

### API Routes

| Route | Methods | Purpose |
|---|---|---|
| `/api/mcp/registry` | GET | List catalog (optional `?category=` / `?q=`). Seeds lazily. Auth optional (returns empty for anonymous). |
| `/api/mcp/installations` | GET / POST / PATCH / DELETE | List (masked secrets) / install / toggle enabled+require_approval / uninstall. Auth required. |
| `/api/mcp/tools` | POST `{slug, tool, args}` | Execute a tool on a server-side MCP connection. Decrypts config, connects, calls, closes. Auth required. |

### Secret Encryption

Connector config values typed `'secret'` in `config_schema` are encrypted with AES-256-GCM before storage and decrypted server-side at runtime. The `maskConfig()` function replaces secret values with `'__set__'` for GET responses — raw secrets are never returned to the client. On re-save, fields arriving as `'__set__'` preserve the existing encrypted value (no wipe).

### Marketplace UI (`/connectors`)

`app/connectors/page.tsx` — full marketplace: Browse / Installed tab toggle, category filter chips, search, `ConnectorCard` grid (from `GET /api/mcp/registry`), `ConnectorDetail` modal (config form with text + password inputs, Install / Save / Uninstall), `InstalledList` (enable/disable toggle, require-approval toggle). The Contractors Room built-in card and the "Embed Lucy" snippet panel are preserved.

Components:
- `components/connectors/ConnectorCard.tsx` — icon, name, category, tools count, Install / Installed / Configure action
- `components/connectors/ConnectorDetail.tsx` — full detail view with config-schema form; secrets show `••• set` placeholder when already set
- `components/connectors/InstalledList.tsx` — per-install enable/disable, require-approval, Configure, Uninstall

### Chat Tool-Use Loop

`app/api/chat/route.ts` — when tools are present for the user, the chat route runs a bounded tool-use loop (max 5 rounds) instead of a single streaming call:

- **OpenAI-compatible** (OpenAI, Groq, Mistral, xAI, OpenRouter, DeepSeek, local/Ollama): uses `openai` SDK `chat.completions.create` with `tools: toOpenAITools(tools)`, accumulates `tool_calls` deltas, executes each via `callTool` server-side, appends `{ role: 'tool', … }` messages, loops.
- **Anthropic**: uses `@anthropic-ai/sdk` `messages.create` with `tools: toAnthropicTools(tools)`, handles `tool_use` blocks, appends `tool_result` user messages, loops.
- **Google/Gemini**: tool-use loop is skipped; Gemini uses its own native streaming path unchanged.
- **No installed connectors**: existing streaming path runs unchanged — zero performance impact.
- **Approval gating**: if an installation has `require_approval: true`, write-like tool calls (name starts with `create/update/delete/send/write/post`) return `{ error: 'approval required' }` so the model explains it needs approval rather than executing silently.

### SSE Tool Events

The chat route emits two metadata events per tool call (in addition to the existing `memoryCount` event):

```
data: {"metadata":{"tool_call":{"slug":"github","tool":"create_issue","args":{…}}}}

data: {"metadata":{"tool_result":{"slug":"github","tool":"create_issue","ok":true}}}
```

The client's `parseSSEStream` `onMetadata` callback in `app/chat/page.tsx` handles these: `tool_call` events add a pending chip to `useChatStore.toolChips`; `tool_result` events update the chip's `ok` status. `ChatWindow` renders the chips inline above the streaming bubble using color-coded pill badges (gray = pending, green = success, red = failed). Chips are cleared at the start of each new send.

### Tests

| File | Covers |
|---|---|
| `__tests__/lib/mcp/secret.test.ts` | Round-trip encrypt/decrypt, random IV, tamper resistance |
| `__tests__/lib/mcp/installer.test.ts` | `validateConfig` (required fields), `maskConfig` (secret → marker, text preserved) |
| `__tests__/lib/mcp/tool-format.test.ts` | `toOpenAITools` / `toAnthropicTools` namespacing + shape |

## Voice (STT + TTS)

Lucy has a built-in voice layer: microphone input (Speech-to-Text) and read-aloud output (Text-to-Speech). **Spec:** `docs/superpowers/specs/2026-06-10-voice-stt-tts-design.md`.

### Config — `useSettingsStore().voice`

Persisted to `lucy:voice` in localStorage. Shape:

```typescript
voice: {
  stt: {
    enabled: boolean;
    provider: 'browser' | 'openai' | 'deepgram' | 'local';
    language?: string;   // BCP-47, e.g. 'en-US'
    baseUrl?: string;    // for openai / local
    model?: string;      // e.g. 'whisper-1'
  };
  tts: {
    enabled: boolean;
    provider: 'browser' | 'openai' | 'local';
    voice: string;       // voice name or 'default'
    speed: number;       // 0.5–2, default 1
    autoRead: boolean;   // auto-read each completed assistant reply
    baseUrl?: string;    // for openai / local
    model?: string;      // e.g. 'tts-1'
  };
  deepgramKey?: string;  // stored client-side (only Deepgram needs its own key)
}
```

### `lib/voice/` Modules

| File | Purpose |
|---|---|
| `types.ts` | `VoiceConfig`, `SttProvider`, `TtsProvider`, `VoiceOption` |
| `stt.ts` | `createSttSession(opts) → SttSession \| null` — browser path uses Web Speech API (live interim + final); cloud providers (`openai`, `deepgram`, `local`) record via `MediaRecorder`, POST audio to `/api/voice/transcribe`. Also exports `sttSupported()`, `recordingSupported()`. |
| `tts.ts` | `speak(text, opts)` / `stopSpeaking()` — browser path uses `speechSynthesis`; cloud path POSTs to `/api/voice/speak` and plays the returned audio blob. Also exports `listVoices()`, `waitForVoices()`, `ttsSupported()`. |

### API Routes

Both routes authenticate via `resolveMemoryAuth` (Supabase session or Lucy API key). Keys are read from **request headers only** — never the body.

**`POST /api/voice/transcribe`** (`app/api/voice/transcribe/route.ts`)
- Accepts `multipart/form-data`: `file` (audio blob), `provider`, `model?`, `language?`, `baseUrl?`.
- Headers: `x-openai-key` (openai / local), `x-deepgram-key` (deepgram). Falls back to `OPENAI_API_KEY` env var.
- Providers: `openai`/`local` → OpenAI SDK `audio.transcriptions.create` (Whisper); `deepgram` → direct `api.deepgram.com/v1/listen` REST call.
- Returns `{ text: string }`.

**`POST /api/voice/speak`** (`app/api/voice/speak/route.ts`)
- Accepts JSON: `{ text, provider, voice?, speed?, model?, baseUrl? }`.
- Header: `x-openai-key`. Falls back to `OPENAI_API_KEY` env var.
- Providers: `openai`/`local` → OpenAI SDK `audio.speech.create` (TTS-1). Returns raw `audio/mpeg` bytes.

### UI

- **Settings page** — `/settings/voice` (`app/settings/voice/page.tsx`): STT card (provider, language, base URL, model) + TTS card (provider, voice selector, speed, auto-read, base URL, model). Reachable via the **Mic** icon in `SettingsNav`.
- **Mic button** in `components/chat/ChatInput.tsx`: browser provider → live Web Speech (interim text appears in input as you speak); cloud providers → press to record, release/stop to transcribe.
- **Read-aloud** on assistant messages (`components/chat/ChatMessage.tsx`): speaker icon per message; `autoRead` flag in TTS config triggers `speak()` automatically when a reply completes.

### Providers

| Provider | Key needed | Notes |
|---|---|---|
| Browser (STT) | None | Web Speech API; Chrome/Edge only; live interim results |
| Browser (TTS) | None | `speechSynthesis`; voices depend on OS |
| OpenAI (STT + TTS) | User's OpenAI key (Settings → Providers) | Whisper STT; TTS-1 / TTS-1-HD |
| Deepgram (STT) | Deepgram key in Settings → Voice | `deepgramKey` stored in voice config; `nova-2` model |
| Local (STT + TTS) | None (or key for private servers) | Any OpenAI-compatible endpoint via `baseUrl` |

---
> Source: [JustLucyHQ/lucy](https://github.com/JustLucyHQ/lucy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
