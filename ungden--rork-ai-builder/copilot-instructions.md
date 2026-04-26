## rork-ai-builder

> Build and ship a **Rork-like AI Mobile App Builder** — a web tool where users describe a mobile app in natural language, an AI agent autonomously builds complete Expo/React Native code, and users can preview, edit, export, sync to GitHub, and build for app stores. Deployed on **Vercel** connected to **Supabase**.

# CLAUDE.md — Rork AI Mobile App Builder

## Goal

Build and ship a **Rork-like AI Mobile App Builder** — a web tool where users describe a mobile app in natural language, an AI agent autonomously builds complete Expo/React Native code, and users can preview, edit, export, sync to GitHub, and build for app stores. Deployed on **Vercel** connected to **Supabase**.

---

## Instructions

- **Repo**: `https://github.com/ungden/rork-ai-builder.git` — cloned at `/Users/alexle/Documents/mobileai/rork-ai-builder`
- **Never rewrite from scratch** — always read existing code first, identify gaps, fix precisely
- **Always read files before editing**
- **Stack**: Next.js 16 (Turbopack), Tailwind v3 (downgraded from v4), Supabase SSR, `@ai-engine/core` workspace package, pnpm + Turborepo monorepo
- **AI providers**: Gemini (`GEMINI_API_KEY`) is the default model; Claude (`ANTHROPIC_API_KEY`) optional for full 11-tool `RorkAgent` loop
- **Deployment**: Vercel (connected to `ungden/rork-ai-builder` GitHub repo), Supabase project ID `wpfufmolfgcjnkxjgpqt`
- **Design direction**: Dark theme, clean/minimal like rork.com — centered text, proper spacing, Inter font
- **Do NOT commit `.env.local`** — credentials go in Vercel dashboard env vars only
- **`vercel.json`** configured with `buildCommand: "pnpm build:web"`, `outputDirectory: "apps/web/.next"`, `maxDuration: 300` for agent routes
- **Supabase env vars on Vercel**: `NEXT_PUBLIC_SUPABASE_URL` = `https://wpfufmolfgcjnkxjgpqt.supabase.co`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `GEMINI_API_KEY` — all confirmed set
- **Supabase schema**: Deployed to live project (user confirmed)
- **Signup**: Only email + password (Name field removed)
- **Next.js 16 uses `proxy.ts`** not `middleware.ts` — file is `src/proxy.ts` with `export async function proxy`

---

## Key Architecture

### How the Landing → Editor Flow Works
1. `HeroPromptBox` (landing page) saves prompt to `sessionStorage('rork_pending_prompt')`
2. If logged in: creates project via `/api/projects` → redirects to `/editor/{id}`
3. If not logged in: redirects to `/signup`, after auth → dashboard → `PendingPromptHandler` auto-creates project → redirects to editor
4. Editor picks up prompt from sessionStorage → passes as `initialPrompt` to `ChatPanel`
5. ChatPanel auto-sends prompt to AI agent after 800ms delay (via `useRef` pattern, NOT `useCallback`)

### Preview Panel (Expo Snack SDK)
- Uses **Expo Snack SDK** (`snack-sdk@6.6.0`) to run real Expo code in an iframe (web preview) and on devices via QR code
- `useSnack` hook (`apps/web/src/hooks/useSnack.ts`) manages the Snack session — creates instance, goes online, provides `webPreviewRef`, `webPreviewURL`, `expoURL`, `isBusy`, `connectedClients`
- `SnackPreview` component (`apps/web/src/components/editor/SnackPreview.tsx`) renders the iframe and handles runtime errors + "Fix with AI"
- Editor page wires `project-files-changed` events to `snack.updateFiles()` — files push to Snack in real-time as AI generates them
- Initial project files are synced to Snack on load via `snackSetAllFiles()`
- `isGenerating` is wired to `agentStore.isRunning` (not projectStore)
- Files apply immediately via `addGeneratingFile()` as each SSE event arrives — real-time preview updates

### AI Agent Pipeline
- **Gemini path**: `GeminiProvider.streamCode()` — uses `write_file` tool declaration, multi-turn tool loop (up to 20 rounds), yields file events via SSE
- **Claude path**: `RorkAgent` with 11 tools (create_plan, write_file, patch_file, search_files, verify_project, delete_file, read_file, list_files, run_test, fix_error, complete)
- System prompt is assembled from 5 modules in `packages/ai-engine/src/prompts/`: `expo-sdk54.ts`, `navigation.ts`, `styling.ts`, `components.ts`, `expo-knowledge.ts`
- All prompts target real Expo SDK 52 code: `expo-router` (file-based routing), `@expo/vector-icons` (Ionicons), `react-native-reanimated`, `react-native-safe-area-context`, `expo-image`, `expo-blur`, `expo-haptics`, `AsyncStorage`, CSS `boxShadow`
- `@react-three/fiber` for 3D graphics (Rork Max capability)

### State Management
- `projectStore` (Zustand + Immer): files, messages, activeFile, isGenerating, selectedModel
- `agentStore` (Zustand + Immer): isRunning, phase, plan, files, progress, processEvent

---

## Discoveries / Gotchas

### Tailwind v4 → v3 downgrade
- Tailwind v4's `@theme {}` hex tokens don't support opacity modifiers and its CSS layer ordering broke basic utilities on Vercel
- Downgraded to Tailwind v3 with `tailwind.config.ts`, standard postcss plugin, and `@tailwind` directives

### Supabase URL was wrong on Vercel
- Originally pointed to `ekeicbetveuvazfgmfii.supabase.co` (wrong) — corrected to `wpfufmolfgcjnkxjgpqt.supabase.co`

### Auto-send bug (FIXED in d749dea)
- `handleAgentRun` was wrapped in `useCallback` with `[files]` in deps → stale closure → prompt never sent
- **Fix**: Replaced with `useRef` pattern. `handleAgentRunRef.current` always points to latest function

### Preview `isGenerating` was never true (FIXED)
- PreviewPanel read `projectStore.isGenerating` but ChatPanel never called `setGenerating()`
- **Fix**: Wired to `agentStore.isRunning` instead — building overlay now shows correctly

### Prompt contradictions (FIXED)
- `expo-sdk54.ts` listed `expo-symbols`/`formSheet` as available but system prompt banned them
- `expo-knowledge.ts` showed `formSheet` with detents code example
- Styling/navigation/components files said "SDK 52" instead of "SDK 54"
- **Fix**: Removed contradictions, fixed version refs, added missing patterns (ActivityIndicator, Alert, StatusBar, LinearGradient, RefreshControl, +not-found.tsx, ImagePicker, Context state management), added Snack package allowlist, deduplicated repeated content

### Snack Preview "Connecting..." forever (FIXED)
- **Root cause**: `PRELOADED_DEPS` included `expo-blur`, `expo-image`, `expo-av`, `expo-haptics`, `expo-camera`, `expo-linear-gradient`, `react-native-screens` — but `snack-content`'s `isModulePreloaded()` returns `false` for ALL of them on SDK 54. When declared as Snack dependencies, Snackager attempts to resolve them. If resolution is slow or fails, `State.isBusy()` returns `true`, which blocks `_sendCodeChangesDebounced()` in `updateTransports()` — code never reaches the web player iframe, resulting in permanent "Connecting..." state.
- **Fix**: Two-phase dependency loading:
  - Phase 1: Only declare truly preloaded deps (expo-router, @expo/vector-icons, react-native-safe-area-context, react-native-reanimated, react-native-gesture-handler, @react-native-async-storage/async-storage, expo-constants, expo-font) — `isBusy()` returns `false` immediately, code pushes work
  - Phase 2: After first client connects (via `addStateListener` watching `connectedClients`), add non-preloaded deps from package.json — initial render is already done so isBusy doesn't block anything visible
  - Added 5-second grace period iframe reload (ONCE only, not looping) as fallback for lost CONNECT messages

### Snack Preview White/Blank Screen (FIXED)
- **Root cause**: Snack web player has TWO code paths for loading the entry point (App.tsx):
  1. **expo-router mode**: If `App.tsx` content matches regex `/import.*expo-router\/entry/i`, the runtime creates a virtual `require.context` for the `app/` directory and renders `<ExpoRoot context={ctx} />`. This makes file-based routing work.
  2. **Normal mode**: Otherwise, it renders `App.tsx`'s default export as a plain React component. The `app/` directory files are completely ignored.
- The AI prompt said "renders ExpoRoot or `<Slot />`" but gave NO concrete App.tsx example. The AI was likely generating `export default function App() { ... }` with expo-router imports — which takes the **normal mode** path, rendering a broken component that can't find routes.
- **Fix (prompt)**: All prompt references to App.tsx now explicitly state it MUST contain ONLY `import 'expo-router/entry';` — no default export, no other code.
- **Fix (safety net)**: `useSnack.updateFiles()` now validates App.tsx content — if it doesn't contain the expo-router/entry import but there are `app/` directory files, it auto-injects the correct entry point.
- `expo-router/entry` is a bundled module (preloaded, no Snackager delay) that resolves to a no-op function — its only purpose is as a signal for the Snack runtime regex detector.

---

## Accomplished (All Completed)

- Tailwind v4 → v3 downgrade (CSS works on Vercel)
- All backend API routes working (agent, projects, files, settings, GitHub sync, EAS build, demo generate, auth)
- Supabase client/server setup with proxy.ts middleware
- AI agent pipeline (Claude + Gemini) with tool loop
- All editor components (ChatPanel, CodePanel, PreviewPanel, FileTree, Toolbar, QRPanel, CommandPalette, AgentStatus)
- Dashboard components (ProjectCard, CreateProjectButton, PendingPromptHandler)
- Auth flow (login, signup email+password, OAuth Google/GitHub, callback, signout)
- Interactive landing page prompt box (HeroPromptBox) with step-by-step progress
- Lovable-style build progress UX (PendingPromptHandler shows animated steps)
- Auto-save, keyboard shortcuts, command palette
- Export ZIP, GitHub sync, EAS build config generation
- Demo mode with pre-seeded files
- Supabase schema deployed to live project
- Signup simplified (name field removed)
- Auto-send bug fixed (useRef pattern)
- Preview "No app yet" empty state (instead of infinite spinner)
- Preview building overlay wired to agentStore.isRunning
- Real-time file application during agent SSE streaming
- Agent prompt quality: contradictions removed, missing patterns added, Snack allowlist added
- Error visibility improved (actual error message shown in chat)
- Unused imports cleaned up across all components
- Build passes: 0 errors, 0 TS errors, all 17 routes compile
- **Expo Snack SDK Migration**: Replaced esbuild bundler with Expo Snack SDK for live preview — supports web preview (iframe) AND real device testing via QR code
  - Created `useSnack` hook and `SnackPreview` component
  - Wired editor page to push files to Snack on load and on every AI file update
  - Removed old esbuild bundler (bundler.ts, LivePreview.tsx, /api/bundle, /api/stub routes, esbuild dependency)
- **AI Prompts Rewritten for Real Expo**: All 6 prompt files completely rewritten to generate real Expo code instead of esbuild/web-only code:
  - expo-router file-based routing (was: custom state-based Navigator)
  - @expo/vector-icons Ionicons (was: lucide-react-native)
  - react-native-reanimated with entering/exiting animations (was: Animated from RN only)
  - react-native-safe-area-context (was: constant padding)
  - expo-image with blurhash (was: RN Image)
  - expo-blur BlurView (was: semi-transparent bg workaround)
  - expo-haptics with Platform.OS guard (was: not available)
  - AsyncStorage (was: localStorage wrapper)
  - CSS boxShadow (was: legacy RN shadow props)
  - expo-status-bar, expo-linear-gradient (was: not available)
- **Rork Max capabilities**: `@react-three/fiber` for 3D games and web AR natively.
- **Fix with AI**: Catch runtime errors inside the Snack iframe, displaying them with a button that auto-prompts the AI to fix it.
- Gemini text streaming fixed — text now yields incrementally during the loop (not buffered until end)
- CodePanel close-tab now actually closes tabs (tracks `openTabs` state, not all files)
- agentStore text_delta now appends to existing message instead of creating unbounded new messages
- generatingFiles cleared properly after agent run (ChatPanel calls `setGenerating(false)`)
- HeroPromptBox catch-all error now redirects to /dashboard instead of always /signup
- runChecks `any` type check uses regex word boundary instead of false-positive substring match
- Unused imports removed: ThinkingLevel (gemini.ts), ParsedFile (tools/index.ts), STEP_LABELS (HeroPromptBox)
- **Snack SDK Lazy Init** (`6064bd6`): Preview no longer connects on mount. `goOnline()` called only when project has >3 files or after AI agent completes. Shows "Waiting for code..." vs "Connecting to Expo..." states via `hasRequestedOnline`.
- **Gemini Tool Loop Resilience** (`6064bd6`):
  - Context compression: fresh chat session after every 6 files to prevent context bloat (was: linear history growth → context exhaustion after ~11 files)
  - Empty response tracking: resets session after 2 consecutive empties, gracefully completes after 4 total empties
  - Retryable API errors: retries timeout/503/rate-limit up to 2 times with 3s delay (was: immediate failure)
  - Reduced plan size from 15-20 to 10-15 files, increased batch from 2-3 to 3-5 files per response
  - Added `[gemini]` logging for each loop iteration
- **App.tsx expo-router/entry Fix**: White screen caused by Snack runtime's dual code path — App.tsx must contain `import 'expo-router/entry';` for expo-router mode. AI prompts updated with explicit instruction. `useSnack.updateFiles()` now auto-injects correct entry point as safety net.

## Current State

- **Live site works end-to-end**: Landing prompt → signup → project created → editor → AI agent builds app → preview shows result
- **Default model**: Gemini 3.1 Pro Preview
- **Preview**: Expo Snack SDK powers live preview (web iframe + QR code for real device testing via Expo Go). Lazy init — connects only when code is ready.
- **AI Prompts**: Fully rewritten for real Expo code — expo-router, @expo/vector-icons, reanimated, safe-area-context, expo-image, expo-blur, expo-haptics, AsyncStorage, CSS boxShadow
- **Old esbuild bundler removed**: LivePreview, bundler.ts, /api/bundle, /api/stub all deleted
- **Rork Max Upgrade**: 3D Games/AR via `@react-three/fiber` and updated AI persona
- **Template packs**: 6 starter templates (Task Manager, Weather Dashboard, Fitness Tracker, Recipe Book, Expense Tracker, Smart Notes) with category filtering on dashboard. Selecting a template creates a project and auto-sends a detailed AI prompt.
- **Conversation persistence**: Agent API saves messages to DB; editor restores them on reload
- **Gemini resilience**: Context compression + empty response recovery + API retry ensures all plan files get written
- **Latest tested**: Build passes, type-check passes, all 18 routes compile

## Remaining Work (Not Started)

- Configure OAuth providers (Google/GitHub) in Supabase dashboard (user action)
- Test Snack SDK lazy init + Gemini resilience end-to-end on production with AI-generated Expo apps
- (Low priority / future) Add `ANTHROPIC_API_KEY` to Vercel for optional Claude model support — Gemini is the default and only active model

---

## Key Files

### Config / Infrastructure
- `vercel.json` — Vercel monorepo build config
- `apps/web/tailwind.config.ts` — Tailwind v3 config with custom colors, fonts, animations
- `apps/web/src/proxy.ts` — Next.js 16 proxy (middleware) for auth
- `supabase/schema.sql` — full Postgres schema

### Pages
- `apps/web/src/app/page.tsx` — landing page
- `apps/web/src/app/(auth)/login/page.tsx` / `signup/page.tsx` — auth
- `apps/web/src/app/dashboard/page.tsx` — projects list + PendingPromptHandler
- `apps/web/src/app/editor/[projectId]/page.tsx` — main editor

### Core Components
- `apps/web/src/components/editor/ChatPanel.tsx` — chat + agent SSE streaming + auto-send
- `apps/web/src/components/editor/PreviewPanel.tsx` — preview container with device frame + Snack status
- `apps/web/src/components/editor/SnackPreview.tsx` — Snack iframe + runtime error handling + Fix with AI
- `apps/web/src/hooks/useSnack.ts` — Snack SDK session management hook
- `apps/web/src/components/landing/HeroPromptBox.tsx` — landing prompt box
- `apps/web/src/components/dashboard/PendingPromptHandler.tsx` — auto-creates project from saved prompt
- `apps/web/src/components/dashboard/TemplateGrid.tsx` — template browsing UI with category filters
- `apps/web/src/lib/templates.ts` — template data (6 templates with prompts, icons, categories)

### Stores
- `apps/web/src/stores/projectStore.ts` — project state (files, messages, selectedModel='gemini')
- `apps/web/src/stores/agentStore.ts` — agent state (phase, plan, files, progress)

### API Routes
- `apps/web/src/app/api/agent/run/route.ts` — agent endpoint (Gemini/Claude, SSE streaming, tool executor)
- `apps/web/src/app/api/projects/route.ts` — GET/POST projects
- `apps/web/src/app/api/projects/[id]/files/route.ts` — GET/PUT files

### AI Engine
- `packages/ai-engine/src/providers/gemini.ts` — GeminiProvider (write_file tool loop)
- `packages/ai-engine/src/providers/claude.ts` — ClaudeProvider
- `packages/ai-engine/src/agent.ts` — RorkAgent (11-tool agentic loop)
- `packages/ai-engine/src/prompts/index.ts` — assembled FULL_SYSTEM_PROMPT
- `packages/ai-engine/src/prompts/expo-sdk54.ts` — SDK rules, library preferences, project structure
- `packages/ai-engine/src/prompts/navigation.ts` — Tabs, Stack, Link, modals, route patterns
- `packages/ai-engine/src/prompts/styling.ts` — StyleSheet, shadows, typography, colors, animations
- `packages/ai-engine/src/prompts/components.ts` — Ionicons, expo-image, expo-av, BlurView, controls
- `packages/ai-engine/src/prompts/expo-knowledge.ts` — Common UI patterns, data fetching, storage, Snack allowlist

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ungden) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
