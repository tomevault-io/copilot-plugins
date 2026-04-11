## sighted

> Sighted 75 is a NextJs app for practicing LeetCode-style coding questions (the "Blind 75" set). It features an in-browser code editor (CodeMirror 6) with 31 selectable syntax themes, JavaScript/Python execution, Go/Java/Rust/C/C++ scaffold-only support, completion tracking via IndexedDB, a multi-palette dark/light theme system with optional editor-theme-driven app theming, a distraction-free Zen Mode, end-to-end encrypted chat (ECDH P-256 + AES-256-GCM), and collaborative Groups with timed coding sessions.

# Sighted 75 — Agent Guidelines

## Project Overview

Sighted 75 is a NextJs app for practicing LeetCode-style coding questions (the "Blind 75" set). It features an in-browser code editor (CodeMirror 6) with 31 selectable syntax themes, JavaScript/Python execution, Go/Java/Rust/C/C++ scaffold-only support, completion tracking via IndexedDB, a multi-palette dark/light theme system with optional editor-theme-driven app theming, a distraction-free Zen Mode, end-to-end encrypted chat (ECDH P-256 + AES-256-GCM), and collaborative Groups with timed coding sessions.

## Tech Stack

- **Runtime:** Node 22+ (use `nvm use 22`)
- **Package manager:** Bun
- **Framework:** Next.js 16 (App Router, Turbopack) + React 19 + TypeScript
- **Styling:** Tailwind CSS v4 (utility-first, CSS custom properties for theme tokens)
- **State management:** Zustand 5
- **Auth:** Custom username/password — `jose` (JWT sessions) + `bcryptjs` (password hashing), no third-party auth library
- **Database:** Neon Serverless Postgres (`@neondatabase/serverless`) + Drizzle ORM (`drizzle-orm`)
- **Editor:** CodeMirror 6 + `@uiw/codemirror-themes-all` (31 syntax themes)
- **Icons:** lucide-react (tree-shakable SVG icons)
- **Persistence:** IndexedDB via `idb-keyval` (local/guest), Neon Postgres (authenticated users)
- **Linter:** oxlint
- **Fonts:** JetBrains Mono (display/code) + IBM Plex Sans (body) via `next/font/google`

## Commands

```bash
bun install        # install dependencies
bun run dev        # start dev server (localhost:3000)
bun run build      # production build
bun run start      # start production server
bun run lint       # lint with oxlint
```

## Architecture

```
app/
├── layout.tsx                    # Root layout (server component): <html>, <body>, fonts, metadata
├── page.tsx                      # Single page: "use client" wrapper rendering <App />
├── globals.css                   # Tailwind directives + theme tokens (6 variants) + keyframes + base styles
├── actions/
│   ├── auth.ts                   # Server Actions: signUp, signIn, signOut (username/password)
│   ├── progress.ts               # Server Actions: getProgress, syncProgress, bulkSyncProgress, saveSettings, etc.
│   ├── chat.ts                   # Server Actions: createRoom, sendMessage, getMessages, listRooms, etc.
│   └── groups.ts                 # Server Actions: createGroup, inviteToGroup, startGroupSession, submitSessionResult, etc.
├── api/
│   ├── auth/
│   │   └── session/
│   │       └── route.ts          # GET /api/auth/session — returns current user from JWT cookie (used by authStore)
│   └── chat/
│       ├── sse/
│       │   └── route.ts          # GET /api/chat/sse — Server-Sent Events for real-time chat messages, group invitations, and session changes
│       └── purge/
│           └── route.ts          # POST /api/chat/purge — purge expired chat rooms
proxy.ts                          # Next.js Proxy: optimistic cookie check for protected routes
src/
├── App.tsx                       # Root client component, hydrates stores, orchestrates layout + group session integration
├── components/                   # Client components (all marked "use client")
│   ├── AuthForms.tsx             # Sign-in / sign-up form (username + password), calls Server Actions
│   ├── CodeEditor.tsx            # CodeMirror 6 wrapper with 31 selectable themes, dynamic theming + user-configurable extensions
│   ├── HintPanel.tsx             # Progressive hint reveal
│   ├── LanguageSelector.tsx      # Language tab selector (JS/Python/Go/Java active; Rust/C/C++ scaffolded)
│   ├── OutputPanel.tsx           # Code execution output display
│   ├── ProgressBar.tsx           # Completion progress indicator
│   ├── QuestionCard.tsx          # Question display with numbered title + examples (optional onToggleComplete for session mode)
│   ├── QuestionsModal.tsx        # All-questions table modal with filters/sort
│   ├── SettingsModal.tsx         # Tabbed settings modal (Editor / Appearance / Account)
│   ├── Timer.tsx                 # Per-question stopwatch timer
│   ├── WelcomeGate.tsx           # First-visit gate: sign in, sign up, or continue locally (guest)
│   ├── ChatModal.tsx             # Right-side resizable drawer for encrypted chat
│   ├── ChatPanel.tsx             # Chat room view: message list + input
│   ├── ChatRoomList.tsx          # Room list sidebar inside ChatModal
│   ├── ChatMessage.tsx           # Individual chat message bubble
│   ├── ChatInput.tsx             # Chat message input field
│   ├── GroupsModal.tsx           # Right-side resizable drawer for groups management
│   ├── GroupDetail.tsx            # Group detail view: tabs (Members / Board / Chat), invite modal, start session, join session banner
│   ├── GroupSession.tsx           # GroupSessionBanner (thin bar in main UI during active session) + SessionResultsModal
│   └── GroupBoard.tsx             # Group stats: member stats table + session history
├── lib/
│   ├── auth/
│   │   ├── session.ts            # JWT encrypt/decrypt via jose, cookie create/delete/get (server-only)
│   │   └── server.ts             # verifySession() / requireSession() helpers (server-only)
│   ├── crypto/
│   │   └── e2e.ts                # E2E encryption primitives: ECDH P-256 key gen, AES-256-GCM encrypt/decrypt, HKDF-SHA-256, key wrapping
│   └── db/
│       ├── index.ts              # Drizzle client (neon-http driver)
│       └── schema.ts             # Drizzle schema: users, sessions, userProgress, userSettings, chat tables, group tables
├── store/                        # Zustand stores + IndexedDB layer (all "use client")
│   ├── themeStore.ts             # Theme mode (dark/light) + palette (emerald/ocean/amber)
│   ├── editorStore.ts            # Editor preferences (font, indentation, feature toggles, editorTheme, adaptAppTheme, zenFullscreen)
│   ├── completionStore.ts        # Question completion tracking
│   ├── questionStore.ts          # Current question selection + randomQuestion
│   ├── codeRunnerStore.ts        # Code execution state
│   ├── authStore.ts              # Auth state: fetches /api/auth/session on hydrate, exposes user/isAuthenticated
│   ├── chatStore.ts              # Chat state: rooms, messages, SSE connection, E2E encryption/decryption, unencrypted group room support
│   ├── chatCryptoStore.ts        # ECDH key pair management (IndexedDB), DM key derivation, room key unwrapping
│   ├── groupStore.ts             # Groups state: groups, invitations, group detail, active/live session, session results
│   ├── timerStore.ts             # Per-question timer state
│   └── db.ts                     # IndexedDB persistence functions
├── runners/                      # Language-specific code execution (all "use client")
│   ├── javaRunner.ts             # Stub — Java not supported in browser (like Go)
│   ├── jsRunner.ts               # In-browser JS execution via Function()
│   ├── pythonRunner.ts           # In-browser Python via Pyodide (lazy-loaded)
│   └── goRunner.ts               # Stub — Go not supported in browser
├── themes/
│   └── editorThemeColors.ts      # Maps each editor theme to app CSS variables for "Adapt App Theme" feature
├── data/
│   └── questions.json            # Question bank (75 Blind questions with keywords + 7-language scaffolds)
└── types/
    └── question.ts               # TypeScript interfaces
```

## Conventions

### Next.js App Router

The app uses Next.js App Router with a hybrid rendering strategy:
- **Server components:** `app/layout.tsx` handles `<html>`, `<body>`, font loading, and metadata.
- **Client components:** Everything under `src/` is marked `"use client"` since the app relies heavily on browser APIs (IndexedDB, CodeMirror, Pyodide, Fullscreen API).
- **Server Actions:** `app/actions/auth.ts` (sign up/in/out), `app/actions/progress.ts` (CRUD for user progress and settings), `app/actions/chat.ts` (room management, messaging, public keys), and `app/actions/groups.ts` (group CRUD, invitations, sessions, results). All Server Actions use `"use server"` and validate sessions via `verifySession()`.
- **Route Handlers:** `app/api/auth/session/route.ts` (GET — client-side session hydration), `app/api/chat/sse/route.ts` (GET — Server-Sent Events for real-time chat, group invitations, and session changes), `app/api/chat/purge/route.ts` (POST — purge expired chat rooms).
- **Path alias:** `@/*` maps to `./src/*` (configured in `tsconfig.json`).

### State Management

All application state lives in Zustand stores under `src/store/`. Do **not** use React Context or custom hooks for shared state. Each store exposes a `hydrate()` method for async initialization from IndexedDB.

Selector pattern — always use granular selectors to avoid unnecessary re-renders:

```tsx
const mode = useThemeStore((s) => s.theme.mode);  // good
const store = useThemeStore();                      // bad — subscribes to everything
```

### Components

- Components are functional, never class-based.
- Components receive data via props or Zustand selectors — no prop drilling of store actions through multiple layers.
- Use `useCallback` for event handlers passed as props. Use functional `setState` updates to avoid stale closures.
- Use ternary (`? :`) instead of `&&` for conditional rendering when the condition could be falsy (0, NaN).
- Hoist static JSX (SVG icons, skeletons) outside components as module-level constants.

### Theming

The theme system uses CSS custom properties with `[data-mode][data-palette]` attribute selectors on `<html>`. There are 3 palettes (emerald, ocean, amber) x 2 modes (dark, light) = 6 variants. Theme tokens are defined in `app/globals.css`. The `themeStore` applies attributes to the DOM and persists to IndexedDB.

**Editor themes:** CodeMirror supports 31 syntax themes via `@uiw/codemirror-themes-all` (Dracula, Nord, Monokai, etc.) plus an "auto" mode that reads CSS variables at editor creation time via `readCSSVar()` in `CodeEditor.tsx`.

**Adapt App Theme:** When enabled in Editor settings, the entire app's CSS variables are overridden to match the selected editor theme's colors (via inline styles on `<html>`). The color map lives in `src/themes/editorThemeColors.ts`. When active, the Appearance tab's mode/palette controls are disabled with an info notice.

### Zen Mode

Zen Mode strips the UI down to just the code editor filling the viewport. Activated via the eye icon in the topbar, exited via Escape or the floating exit button. An optional "Fullscreen" toggle in Appearance settings uses the browser Fullscreen API for true fullscreen. Exiting fullscreen (via browser chrome or Esc) also exits zen mode.

### Chat (E2E Encrypted)

The app includes an end-to-end encrypted chat system for authenticated users. Opened via the message icon in the topbar; renders as a right-side resizable drawer (`ChatModal`).

**Encryption:** ECDH P-256 key pairs are generated client-side and stored in IndexedDB (`chatCryptoStore`). DM rooms derive a shared AES-256-GCM key via ECDH + HKDF-SHA-256. Group chat rooms created by the Groups feature are **unencrypted** (plaintext stored in the `ciphertext` column with `iv = "none"`) since members join dynamically and proper key distribution would be too complex. The `chatStore` detects unencrypted group rooms via `isUnencryptedGroupRoom()` and skips encryption/decryption for them.

**Real-time:** `app/api/chat/sse/route.ts` provides a Server-Sent Events endpoint. The client connects on chat hydration and receives `new_message`, `group_invitation`, and `group_session_changed` events. Polling intervals check for new messages, invitation count changes, and active session status changes.

**Key files:** `src/lib/crypto/e2e.ts` (primitives), `src/store/chatCryptoStore.ts` (key management), `src/store/chatStore.ts` (rooms, messages, SSE), `app/actions/chat.ts` (server actions), `src/components/Chat*.tsx` (UI).

### Groups

Groups enable collaborative coding sessions. Authenticated users can create up to 5 groups and invite others by username. Opened via the Users icon in the topbar; renders as a right-side resizable drawer (`GroupsModal`).

**Group sessions** are the core feature: the admin picks a question and timer duration, then starts a session. All members can join the session, which **takes over the main app view** — the same code editor, question card, and output panel the user normally works with. The session does **not** open a separate editor. Instead:

1. `groupStore.liveSession` is set when a user joins (via `joinSession()`).
2. `App.tsx` reads `liveSession` and overrides the displayed question to the session's question.
3. A `GroupSessionBanner` renders below the main header showing: group name, question, countdown timer, submission count, Submit/End/Leave controls.
4. The "All Questions", "Zen Mode", "Random Question", and "Mark Complete" buttons are hidden during a session since the question is locked.
5. Submit calls `submitResult()` on the group store instead of saving locally.
6. When the timer expires or the admin ends the session, a `SessionResultsModal` pops up showing the results table.

**Invitations:** Users receive invitations visible in the Groups drawer. Accepting adds the user to the group and its auto-created chat room. SSE delivers `group_invitation` events for real-time notification badges.

**Key files:** `app/actions/groups.ts` (13 server actions), `src/store/groupStore.ts` (state + `liveSession`), `src/components/GroupsModal.tsx` (drawer), `src/components/GroupDetail.tsx` (detail view + join banner), `src/components/GroupSession.tsx` (banner + results modal), `src/components/GroupBoard.tsx` (stats).

### CSS / Tailwind

- All styles use Tailwind CSS v4 utility classes applied directly in component JSX.
- Theme tokens (CSS custom properties) are defined in `app/globals.css` under `[data-mode][data-palette]` selectors.
- Reference theme variables via Tailwind arbitrary values: `bg-[var(--bg-primary)]`, `text-[var(--accent)]`.
- Custom `@theme` block in `app/globals.css` defines font families, border radii, and animation keyframes.
- Complex pseudo-element styles (toggle switches, range sliders, CodeMirror overrides) remain as plain CSS in `app/globals.css`.
- Never use `transition: all` — list properties explicitly via Tailwind's `transition-[prop]` syntax.
- Honor `prefers-reduced-motion` (handled in globals.css base styles).
- Use `overscroll-contain` on modals/overlays.
- Use `tabular-nums` for numeric displays.
- Use `text-balance` on headings.

### Accessibility

- Icon-only buttons must have `aria-label`.
- Decorative SVGs must have `aria-hidden="true"`.
- All interactive elements must have visible `focus-visible` states.
- Modals must close on Escape and overlay click.

### Authentication

The app uses a custom username/password auth system with no third-party auth library. There is no email — only a unique username and a secure password.

**How it works:**

1. **Sign up / Sign in** — `app/actions/auth.ts` exposes `signUp()`, `signIn()`, `signOut()` Server Actions. Passwords are hashed with `bcryptjs` (cost 12). Usernames are lowercased, unique, and restricted to `[a-zA-Z0-9_.-]` (3–30 chars).
2. **Sessions** — Stateless JWT tokens signed with `HS256` via `jose`. The JWT payload contains `{userId, username, displayName, expiresAt}`. Stored in an `httpOnly` cookie named `sighted75:session` with 30-day expiry.
3. **Server-side checks** — `src/lib/auth/server.ts` exports `verifySession()` (returns payload or `null`) and `requireSession()` (throws if unauthenticated). Used by Server Actions in `app/actions/progress.ts`.
4. **Client-side hydration** — `authStore` fetches `GET /api/auth/session` on mount to populate `user` / `isAuthenticated` state. No auth client library is used.
5. **Proxy** — `proxy.ts` performs an optimistic cookie-presence check for protected routes (currently `/account/*`).
6. **Guest mode** — Users can skip auth entirely via "Continue Locally" on the `WelcomeGate`. Progress is stored in IndexedDB only. The `localStorage` key `sighted75:auth-choice` tracks whether the user chose `"local"` or `"authenticated"`.

**Environment variables:** `AUTH_SECRET` (HS256 signing key), `DATABASE_URL` (Neon Postgres connection string).

**Do not** introduce a third-party auth library (NextAuth, Better Auth, Clerk, etc.). Keep auth self-contained in the files listed above.

### Database

Neon Serverless Postgres via `@neondatabase/serverless` + Drizzle ORM. Schema lives in `src/lib/db/schema.ts`.

**Tables:**

| Table | Purpose |
|-------|---------|
| `users` | `id` (text PK, UUID), `username` (unique), `password_hash`, `display_name`, `created_at` |
| `sessions` | `id` (text PK), `user_id` FK → users, `expires_at`, `created_at` (reserved for future DB-backed sessions) |
| `user_progress` | Per-user per-question progress: completed, attempts, revisits, best solution, language, time spent. Unique index on `(user_id, question_id)`. |
| `user_settings` | Per-user JSON blobs for theme and editor preferences. `user_id` PK FK → users. |
| `user_public_keys` | ECDH P-256 public keys for E2E chat encryption. `user_id` PK FK → users. |
| `chat_rooms` | Chat rooms with type (`dm` or `group`), optional `encrypted_room_key`, `expires_at`. |
| `chat_room_members` | Room membership with per-member `encrypted_room_key`. Composite PK `(room_id, user_id)`. |
| `chat_messages` | Encrypted messages: `ciphertext`, `iv`, `sender_id`, `room_id`, `created_at`. |
| `groups` | Group identity: `name`, `created_by`, linked `chat_room_id`, timestamps. |
| `group_members` | Group membership with `admin`/`member` role. Composite PK `(group_id, user_id)`. |
| `group_invitations` | Pending/accepted/declined invitations. Unique index on `(group_id, invitee_id)`. |
| `group_sessions` | Timed coding sessions: `question_id`, `duration_secs`, `status` (active/completed), `started_at`. |
| `group_session_results` | Per-user submission results per session: `completed`, `time_spent_secs`, `code`. Composite PK `(session_id, user_id)`. |

**Schema changes:** Use `bunx drizzle-kit push` to push schema changes to Neon (no migration files). Config in `drizzle.config.ts`.

### IndexedDB Keys

All keys are prefixed with `sighted75:` — `sighted75:completed`, `sighted75:theme`, `sighted75:editor` (includes editorTheme, adaptAppTheme, zenFullscreen), `sighted75:currentQuestion`, `sighted75:solution:{id}`, `sighted75:chat-keypair` (ECDH key pair for E2E chat).

### Typography

- Use `\u2026` (ellipsis character) instead of `...` in UI text.
- Loading states should end with ellipsis: `"Running\u2026"`, `"Loading\u2026"`.

## Skills

**Always consult the skills in `.agents/skills/` before making changes.** These contain binding guidelines for this project:

- **`.agents/skills/vercel-react-best-practices/`** — React performance optimization rules (57 rules across 8 categories). Read `AGENTS.md` in that directory for the full guide. Apply these when writing or refactoring any React code.
- **`.agents/skills/web-design-guidelines/`** — Vercel Web Interface Guidelines for accessibility, animation, forms, typography, and more. Fetch the latest rules from the source URL in `SKILL.md` and audit UI code against them.
- **`.agents/skills/ui-ux-pro-max/`** — Design system generation tool. Use when redesigning UI or creating new visual components. Enforces no-emoji icons (use SVGs), specific font/palette choices, and layout patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ruralad)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ruralad)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
