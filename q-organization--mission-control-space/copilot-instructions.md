## mission-control-space

> The developer primarily uses TTS (text-to-speech / voice dictation) to issue commands. Common misheard words to watch for:

# Mission Control Space

## Voice-to-Text Notice

The developer primarily uses TTS (text-to-speech / voice dictation) to issue commands. Common misheard words to watch for:

| You might hear | Actual meaning |
|----------------|----------------|
| Cloud, Claude | Claude (the AI) |
| model | modal |
| sub-base, subbase | Supabase |
| planet-form | platform |
| no-tion, ocean | Notion |
| way-socket | WebSocket |
| bite, white | Vite |
| super base | Supabase |
| hooks | hooks (React) |
| hurler | Howler (audio lib) |
| foul AI, fall AI, fal | FAL.ai (image generation API) |
| edge function | Supabase Edge Function |
| real time, real-time | Realtime (Supabase feature) |

Always interpret ambiguous words in the context of this project's tech stack before asking for clarification.

---

## What This Project Is

Mission Control Space is a **multiplayer productivity game** where players control spaceships in a 2D space world. The core purpose: players complete real tasks (synced from Notion) and business/product goals by flying to planets and landing on them. Everything is designed for **multiplayer** - the whole point of the game is that players can see each other's progress, evolution, and activity in real time.

### Core Loop
1. Players fly spaceships around a 10,000 x 10,000 pixel space world
2. Planets represent goals and tasks (business milestones, product features, Notion tickets)
3. Landing on a planet completes the task and earns points
4. Points are spent on ship upgrades (size, speed, weapons, visual effects)
5. All progress is visible to other players in real time

### Players & Zones
Each player has a dedicated zone in the world:
- **Quentin** (orange) - East (8000, 5000)
- **Alex** (blue) - Northeast (7100, 2900)
- **Armel** (green) - North (5000, 2000)
- **Solène** (pink) - Northwest (2900, 2900)
- **Hugues** (purple) - West (2000, 5000)

Notion tasks assigned to a player appear as planets in their zone.

---

## Tech Stack

- **Framework**: React 18 + TypeScript
- **Build**: Vite (dev server on port 5175)
- **Rendering**: HTML5 Canvas 2D (custom game engine, no framework)
- **Database & Realtime**: Supabase (PostgreSQL + Realtime subscriptions)
- **Multiplayer Positions**: WebSocket server (Node.js `ws` library)
- **Audio**: Howler.js
- **Animations**: canvas-confetti
- **AI Images**: FAL.ai (nano-banana model)
- **Deployment**: AWS S3 (frontend) + EC2 (WebSocket server)

---

## Available Tools — Use Them Proactively

You have full access to the following tools. **Proactively offer to use them** whenever they can help achieve a goal — don't wait to be asked.

### Supabase CLI
The Supabase CLI (`supabase`) is installed and available. Use it to:
- Create/modify database tables and columns
- Deploy and manage Edge Functions
- Run migrations
- Inspect database state
- Manage realtime publications

If a feature needs a new table, column, Edge Function, or RLS policy — just do it directly via the CLI.

### AWS Deployment (Automatic via GitHub Actions)
Pushing to `main` triggers a GitHub Actions workflow that automatically builds and deploys the frontend to AWS S3. You do NOT need to manually deploy. Just push the code and it ships.

### WebSocket Server Deployment (Manual via SSH)
The WebSocket server runs on EC2 (`13.250.26.247:8080`, Singapore `ap-southeast-1`). The SSH key is at the project root: `mission-control-ws.pem`. Deploy with:
```bash
scp -i mission-control-ws.pem ws-server/server.js ec2-user@13.250.26.247:~/ws-server/
ssh -i mission-control-ws.pem ec2-user@13.250.26.247 "cd ~/ws-server && pm2 restart mission-control-ws"
```
Deploy the WS server **before** pushing frontend changes that depend on new message types.

### FAL.ai Image Generation API
The FAL.ai API key is available in `src/App.tsx` (constant `FAL_API_KEY`). Use it to generate images for:
- New planet visuals
- Ship upgrade artwork
- Custom planet mascots
- Any visual asset the game needs

When working on a feature that could benefit from a generated image (new planet type, new upgrade, new visual element), **offer to generate one** using the FAL.ai API rather than using placeholder graphics.

### Summary
You have the database CLI, automatic frontend deployment, WebSocket server SSH access, and an image generation API. When implementing a feature, use the full toolkit end-to-end — create the DB schema, write the frontend code, deploy WS server changes if needed, generate any needed images, and the frontend deployment happens automatically on push.

---

## Global UI Rules

### No Visible Scrollbars

**Anywhere in the entire application**, if content is scrollable, the scrollbar must be **hidden visually** while still allowing scroll functionality. Apply this CSS pattern globally:

```css
/* Hide scrollbar but keep scroll functionality */
scrollbar-width: none;           /* Firefox */
-ms-overflow-style: none;        /* IE/Edge */
&::-webkit-scrollbar {           /* Chrome/Safari/Opera */
  display: none;
}
```

For inline styles in React:
```tsx
style={{
  overflow: 'auto',           // or 'scroll' — allow scrolling
  scrollbarWidth: 'none',     // Firefox
  msOverflowStyle: 'none',   // IE/Edge
}}
// Also add a CSS class or <style> tag for ::-webkit-scrollbar { display: none }
```

**This applies to every scrollable element** - modals, lists, dashboards, sidebars, dropdowns, panels, etc. Never show a visible scrollbar.

### Minimalist UI Copy
- No description text, helper text, or explanatory sentences unless explicitly asked
- Titles and labels only - keep UI minimal
- No placeholder descriptions or subtitle paragraphs

### Fonts
- **Space Grotesk** - Main UI font (weights 400-700)
- **Orbitron** - Tech/title font (weights 400-700)

### Color Palette
- Background: `#0a0a12`
- Orange (Quentin): `#ffa500`
- Blue (Alex): `#5490ff`
- Green (Armel): `#4ade80`
- Pink (Milya): `#ff6b9d`
- Purple (Hugues): `#8b5cf6`
- Gold (Achievements): `#ffd700`

---

## Multiplayer-First Development

**Every new feature MUST support multiplayer.** This is non-negotiable. The entire game syncs to Supabase so all players see each other's state.

### How Sync Works (Hybrid Architecture)

**High-frequency (positions):** WebSocket server at `ws://13.250.26.247:8080`
- Player positions broadcast every frame (~60Hz)
- Other clients interpolate with LERP (factor 0.15)
- Team-based rooms (players only see their team)

**Persistent data:** Supabase Realtime
- Tables with realtime enabled: `teams`, `players`, `point_transactions`, `notion_planets`
- Any data change triggers live updates to all connected clients
- Player upgrades, points, completed planets all sync via Supabase

### When Adding a New Feature, Ask Yourself:
1. Does this data need to be visible to other players? **If yes → store in Supabase with realtime**
2. Does this need sub-frame updates (positions, animations)? **If yes → use WebSocket**
3. Is this purely local (audio prefs, camera position)? **If yes → localStorage only**

### Key Rule
Never store multiplayer-relevant state only in localStorage. If another player should see it, it must go through Supabase.

---

## Documentation Reference

Read the relevant doc **before** working on any of these areas:

| Area | Document | What it covers |
|------|----------|----------------|
| **Supabase setup** | `MULTIPLAYER_SETUP.md` | Database tables, SQL schemas, RLS policies, environment config, testing procedures |
| **Multiplayer architecture** | `docs/multiplayer-architecture.md` | WebSocket + Supabase hybrid design, position sync, upgrade sync, reconnection handling |
| **Notion integration** | `docs/notion-integration.md` | Full Notion sync system - Edge Functions, webhook flow, planet creation from tasks, priority effects, claiming, completing, reassigning |
| **Deployment** | `docs/deployment.md` | AWS S3 static hosting, IAM setup, GitHub Actions CI/CD, manual deploy steps |
| **Visual design** | `docs/visual-design-guide.md` | Art style, color palettes, AI prompt templates for planet/ship generation |
| **Weapons system** | `docs/WEAPONS.md` | All 4 weapons, physics, damage calculations, code locations |

---

## Supabase Connection

**Read `MULTIPLAYER_SETUP.md` before making any database changes.**

- **Project URL**: `https://qdizfhhsqolvuddoxugj.supabase.co`
- **Client init**: `src/lib/supabase.ts`
- **Env vars**: `.env` file (`VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`)
- **No auth**: Players identified by username + team, no login system
- **Realtime config**: `eventsPerSecond: 100`

### Tables

| Table | Purpose | Realtime |
|-------|---------|----------|
| `teams` | Team data, shared points | Yes |
| `players` | Player profiles, ship upgrades, online status | Yes |
| `ship_positions` | Position data (separate for performance) | No |
| `point_transactions` | Audit log of points earned | Yes |
| `point_config` | Points per task type configuration | No |
| `notion_planets` | Notion tasks synced as game planets | Yes |
| `notion_user_mappings` | Maps Notion users to game players | No |

### Adding a New Table
1. Create the table in Supabase dashboard
2. Add it to `supabase_realtime` publication if other players need live updates
3. Set RLS policies (current pattern: public access, no auth)
4. Add the hook/query in `src/hooks/`
5. Document the schema in `MULTIPLAYER_SETUP.md`

---

## Notion Integration

**Read `docs/notion-integration.md` before modifying anything Notion-related.**

### Flow
1. Notion automation triggers webhook → Supabase Edge Function `notion-webhook`
2. Edge Function syncs task data to `notion_planets` table
3. Realtime subscription pushes update to all clients
4. `useNotionPlanets.ts` hook converts DB rows to game Planet objects
5. Planets appear in the assigned player's zone

### Edge Functions
- `notion-webhook` - Receives Notion automation triggers
- `notion-sync` - Full task synchronization
- `notion-create` - Create Notion pages from in-game
- `notion-claim` - Assign planets to players
- `notion-complete` - Mark tasks as done in Notion
- `notion-delete` - Remove planets

### Priority Visual Effects
- **Critical**: Pulsing red glow + meteor storm animation
- **High**: Lightning bolt strikes
- **Medium/Low**: Standard glow

---

## Key Source Files

| File | Size | Purpose |
|------|------|---------|
| `src/App.tsx` | ~10k lines | Main React component, game state, purchase logic, modal management |
| `src/SpaceGame.ts` | ~8k lines | Canvas game engine, rendering, physics, input, particles, effects |
| `src/SoundManager.ts` | ~19k lines | Howler.js wrapper, all audio management |
| `src/types.ts` | | TypeScript interfaces for game objects |
| `src/lib/supabase.ts` | | Supabase client initialization |
| `src/components/ControlHubDashboard.tsx` | | Business analytics dashboard |
| `src/components/QuickTaskModal.tsx` | | Create new planets in-game |
| `src/components/ReassignTaskModal.tsx` | | Reassign Notion planets to players |

### Hooks

| Hook | Purpose |
|------|---------|
| `useTeam.ts` | Team creation/joining, race condition handling |
| `useMultiplayerSync.ts` | Player registration, realtime subscriptions, online status |
| `usePlayerPositions.ts` | WebSocket connection, position broadcast, interpolation |
| `useNotionPlanets.ts` | Notion planet fetching, realtime sync, complete/claim/reassign |
| `useSupabaseData.ts` | Goals, custom planets, user data, Supabase persistence |
| `usePromptHistory.ts` | AI generation history tracking |

---

## Dev Commands

```bash
npm run dev        # Start Vite dev server (port 5175)
npm run build      # TypeScript check + production build
npm run preview    # Preview production build locally
```

### WebSocket Server (local dev)
```bash
cd ws-server
npm install
npm run dev        # Starts on port 8080
```

---

## Game Physics & Constants

- **World size**: 10,000 x 10,000 pixels
- **Acceleration**: 0.18
- **Max speed**: 7 (14 with boost)
- **Friction**: 0.992
- **Rotation speed**: 0.06
- **Docking distance**: 50 pixels
- **Center**: (5000, 5000) - Black hole / Achievements zone
- **Mission Control**: (5000, 8080) - Shop + Planet Factory

---

## Controls

- **WASD**: Movement
- **Q/E**: Rotation
- **Space**: Thrust
- **C**: Complete planet (when docked)
- **X**: Fire weapon
- **B**: Boost

---

## State Management

**Dual storage**: localStorage for instant access + Supabase for multiplayer persistence.

### localStorage Keys
| Key | Data |
|-----|------|
| `mission-control-player-id` | Local player UUID |
| `mission-control-team-id` | Current team UUID |
| `mission-control-space-state` | Main game state |
| `mission-control-custom-planets` | User-created planets |
| `mission-control-user-ships` | Ship upgrades per user |
| `mission-control-goals` | Milestone goals |
| `mission-control-user-planets` | User's personal planet |
| `mission-control-audio-prefs` | Audio settings (local only) |

Data syncs to Supabase on changes and periodically (every ~5 seconds for positions).

---

## Git Rules

- Pushing directly to `main` is fine for this repo
- No Claude Code footers in commit messages
- Ask before committing - don't auto-commit after code changes
- Check `git status` before any branch operations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Q-organization) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
