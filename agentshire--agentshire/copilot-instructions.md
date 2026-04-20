## agentshire

> > OpenClaw plugin: maps AI Agents as NPCs in a 3D low-poly town.

# Agentshire Plugin Architecture Guide

> OpenClaw plugin: maps AI Agents as NPCs in a 3D low-poly town.
> This file is for AI assistants and developers. User docs: [README.md](README.md).

## Tech Stack

- **Runtime**: OpenClaw Plugin SDK (Node.js / ESM)
- **Frontend**: Three.js + vanilla TypeScript (no framework), Vite build, multi-page entries (game / editor / preview / citizen-editor)
- **Protocol**: WebSocket JSON (plugin ↔ frontend), `AgentEvent` / `GameEvent` discriminated unions
- **Audio**: Web Audio API procedural synthesis (zero audio files for ambient) + mp3 BGM
- **Models**: GLTF/GLB, `@gltf-transform` optimization
- **Testing**: Vitest (root + town-frontend each with independent config)

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            OpenClaw Runtime                              │
│                                                                          │
│   Gateway ─── Agent Sessions ─── Hook System ─── LLM Providers           │
└────────┬──────────────────────────────┬──────────────────────────────────┘
         │ channel.reply                │ 10+ hook callbacks
         ▼                              ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  Plugin Layer (Node.js)                               src/plugin/        │
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   channel    │  │     hook-    │  │   ws-server  │  │    tools     │  │
│  │   inbound    │  │  translator  │  │   WS:55211   │  │  11 AI tools │  │
│  │   dispatch   │  │  Hook→Event  │  │   broadcast  │  │  plan / step │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │     plan-    │  │    editor-   │  │   citizen-   │  │   llm-proxy  │  │
│  │    manager   │  │     serve    │  │  chat-router │  │  2 parallel  │  │
│  │    plan SM   │  │  HTTP:55210  │  │   msg route  │  │  LLM direct  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                    │
│  │   subagent-  │  │   outbound-  │  │    session-  │                    │
│  │    tracker   │  │    adapter   │  │    history   │                    │
│  │  JSONL watch │  │  media→card  │  │  chat reload │                    │
│  └──────────────┘  └──────────────┘  └──────────────┘                    │
│                                                                          │
└───────┬───────────────────┬──────────────────────────────────────────────┘
        │                   │
        │  WS :55211        │  HTTP :55210
        │  AgentEvent(26+)  │  Editor API / Static Assets
        ▼                   ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  Bridge Layer (Browser Bundle)                        src/bridge/        │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────── ──┐  │
│  │                       DirectorBridge                               │  │
│  │                                                                    │  │
│  │   Phase: idle → summoning → assigning → going_to_office            │  │
│  │          → working → publishing → returning ──→ idle               │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    Event-    │  │    Route-    │  │   Citizen-   │  │   implicit-  │  │
│  │  Translator  │  │   Manager    │  │   Manager    │  │     chat     │  │
│  │   fallback   │  │   A* + ack   │  │  spawn seq   │  │  10 scenes   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    State-    │  │   NpcEvent-  │  │   Activity-  │  │   ToolVfx-   │  │
│  │   Tracker    │  │    Queue     │  │    Stream    │  │    Mapper    │  │
│  │  id↔npc map  │  │  bubble prot │  │  500ms flush │  │  tool→anim   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                                          │
└───────┬───────────────────────────────────────────────────────┬──────────┘
        │                                                       │
        │  emit GameEvent (65 types)                GameAction  │
        ▼                                           (14 types)  ▲
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  Frontend Layer (Three.js + TypeScript)         town-frontend/src/       │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐   │
│  │   MainScene ── EventDispatcher (65 types) ── update() loop        │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   workflow/  │  │     npc/     │  │      ui/     │  │    audio/    │  │
│  │ Choreographer│  │  NPC 7-state │  │  ChatBubble  │  │   BGM 4-trk  │  │
│  │ 4 Orchestratr│  │  AgentBrain  │  │   NpcCard    │  │   Ambient    │  │
│  │  ModeManager │  │  DailyBhvr   │  │  MediaView   │  │  WebAudioAP I│  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    scene/    │  │    editor/   │  │    engine/   │  │     data/    │  │
│  │ Town/Office  │  │  TownEditor  │  │    Engine    │  │ GameProtocol │  │
│  │   Museum     │  │   Workshop   │  │  World/Input │  │  TownConfig  │  │
│  │ VehicleMgr   │  │   Preview    │  │    Screen    │  │  CharModels  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

  Shared Contracts (src/contracts/):
    AgentEvent (26+ variants) · AgentPhase · MediaContent · AG-UI Protocol
```

## Directory Structure

```
agentshire/
├── index.ts                       # Plugin entry (hook registration / service startup / nudge)
├── openclaw.plugin.json           # Plugin manifest
├── package.json
│
├── src/
│   ├── plugin/                    # Node.js plugin layer (19 files, see plugin/AGENTS.md)
│   │   ├── channel.ts             # ChannelPlugin implementation
│   │   ├── hook-translator.ts     # Hook → AgentEvent translation
│   │   ├── ws-server.ts           # WebSocket server + session management
│   │   ├── tools.ts               # AI tool registration (11 tools)
│   │   ├── auto-config.ts         # Zero-config auto-create Agent + Binding
│   │   ├── plan-manager.ts        # Multi-agent plan state machine
│   │   ├── citizen-agent-manager.ts # Independent citizen Agent create/disable/update
│   │   ├── citizen-chat-router.ts # User ↔ citizen Agent message routing
│   │   ├── citizen-workshop-manager.ts # Citizen workshop config persistence
│   │   ├── editor-serve.ts        # Editor HTTP API (asset CRUD / GLB optimize / publish)
│   │   ├── llm-proxy.ts           # Lightweight LLM proxy (2 concurrent, anthropic/openai)
│   │   ├── session-history.ts     # Cross-session chat history loading
│   │   ├── soul-prompt-template.ts # Soul file AI generation template
│   │   ├── outbound-adapter.ts    # Outbound message adapter (text+media → deliverable_card)
│   │   ├── subagent-tracker.ts    # Sub-agent session log real-time forwarding
│   │   ├── session-log-watcher.ts # JSONL log incremental reader
│   │   ├── custom-asset-manager.ts # Custom GLB asset management
│   │   ├── town-session.ts        # Session ID normalization
│   │   └── runtime.ts             # Runtime injection
│   │
│   ├── bridge/                    # Bridge layer (12 files, see bridge/AGENTS.md)
│   │   ├── DirectorBridge.ts      # Central orchestrator (Phase state machine)
│   │   ├── EventTranslator.ts     # AgentEvent → GameEvent fallback translation
│   │   ├── RouteManager.ts        # A* pathfinding + move acknowledgment + destination scoring
│   │   ├── CitizenManager.ts      # Citizen spawn animation sequence + persona switch detection
│   │   ├── ActivityStream.ts      # Activity log + thinking stream (500ms flush)
│   │   ├── NpcEventQueue.ts       # Dialog bubble guard period
│   │   ├── StateTracker.ts        # agentId ↔ npcId bidirectional mapping + workstation pool
│   │   ├── ToolVfxMapper.ts       # Tool → VFX/animation/emoji pure functions
│   │   ├── ReconnectManager.ts    # Exponential backoff reconnection
│   │   ├── implicit-chat.ts       # NPC implicit LLM calls (10 scene types, DI injection)
│   │   ├── index.ts               # Barrel export
│   │   └── data/route-config.ts   # Pathfinding graph data
│   │
│   ├── contracts/                 # Shared types
│   │   ├── events.ts              # AgentEvent discriminated union (22+ variants)
│   │   ├── agent-state.ts         # AgentPhase / AgentStateSnapshot
│   │   ├── media.ts               # Multimodal content types
│   │   ├── agui.ts                # AG-UI protocol events
│   │   └── registry.ts            # Project registry
│   │
│   └── town-souls.ts              # Soul file loader (multi-level search + fuzzy matching)
│
├── town-souls/                    # Preset soul files (10: steward + 8 citizens + template)
├── town-workspace/                # Workspace template (copied to user dir on first install)
│
└── town-frontend/                 # 3D Frontend
    └── src/
        ├── main.ts                # Game entry (WS connection / implicit chat proxy)
        ├── main.tsx               # React entry (standalone)
        ├── types.ts               # Shared types (weather / time period / buildings / NPC roles / modes)
        ├── game/                  # Scene management (see game/AGENTS.md)
        │   ├── MainScene.ts       # Main scene (1622 lines, update loop + subsystem orchestration)
        │   ├── EventDispatcher.ts # 65 GameEvent type routing
        │   ├── DialogManager.ts   # Dialog streaming display + work logs
        │   ├── GameClock.ts       # 24h cycle (6 periods, night 3x speed)
        │   ├── DailyScheduler.ts  # NPC daily scheduling + AgentBrain + nightly reflection
        │   ├── WeatherSystem.ts   # 12 weather types + 10 daily themes state machine
        │   ├── SceneBootstrap.ts  # Boot flow (PublishedCitizenConfig loading)
        │   ├── minigame/          # ★ Banwei Buster mini-game (MinigameSlot interface)
        │   ├── workflow/          # Workflow orchestration (Choreographer + 4 Orchestrators)
        │   ├── scene/             # 3D scenes (Town/Office/Museum Builder)
        │   └── visual/            # VFX + camera + lighting + weather particles + asset loading
        │
        ├── npc/                   # NPC system (15 files)
        │   ├── NPC.ts             # 7-state state machine + AnimationMixer crossfade
        │   ├── NPCManager.ts      # NPC container management
        │   ├── AgentBrain.ts      # L1 daily plan / L2 tactical / L3 dialog — 3-tier AI decisions
        │   ├── DailyBehavior.ts   # 9-state daily behavior state machine
        │   ├── CasualEncounter.ts # ★ Zero-LLM lightweight social (preset scripts)
        │   ├── CitizenChatManager.ts # ★ Mayor ↔ citizen real-time chat
        │   ├── DialogueScripts.ts # ★ 400+ preset dialog lines (weather/time context)
        │   ├── RoutineTemplates.ts # ★ 5 behavior templates (matched by specialty)
        │   ├── EncounterManager.ts # LLM deep multi-turn dialog + summary
        │   ├── TownJournal.ts     # Event stream + daily narrative summary
        │   ├── ActivityJournal.ts # Activities / dialogs / relationships / daily plan / reflections
        │   ├── PersonaStore.ts    # Persona cache + compact prompt builder
        │   ├── SpotAllocator.ts   # Anti-overlap spiral search
        │   ├── StatusIndicator.ts # Overhead 3D status indicator
        │   └── FollowBehavior.ts  # Follow behavior
        │
        ├── ui/                    # UI panels (17 files)
        ├── data/                  # Data layer
        │   ├── GameProtocol.ts    # GameEvent (65 types) / GameAction (14 types)
        │   ├── TownConfig.ts      # Town config (v4)
        │   ├── CitizenWorkshopConfig.ts # ★ Citizen workshop config system
        │   ├── CharacterModelRegistry.ts # ★ Three-source character model registry
        │   └── ...                # TownConfigStore / CharacterRoster / DataSource
        │
        ├── editor/                # Town editor (26+ files, 3 entry pages)
        │   ├── main.ts            # Map editor entry
        │   ├── TownEditor.ts      # Editor core (select / drag / group / align / undo)
        │   ├── EditorScene.ts     # Editor 3D scene (1816 lines)
        │   ├── preview-main.ts    # ★ Game-level preview entry
        │   ├── citizen-main.ts    # ★ Citizen workshop entry
        │   └── citizen/           # ★ Citizen workshop subsystem (Workshop/Stage/ModelPicker/SoulEditor/AnimMapping)
        │
        ├── audio/                 # BGMManager + AmbientSoundManager + AudioSystem
        ├── engine/                # Engine / World / Input / Screen / Performance
        ├── narrative/             # NarrativeEngine + demo sequences (8 acts)
        ├── platform/              # PlatformBridge (iframe communication)
        ├── hooks/                 # useAgents / useWebSocket
        └── utils/                 # Filters / Math / RingBuffer / command-parser
```

## Data Flow

### Main Workflow

```
┌──────┐  text   ┌──────────┐  WS   ┌──────────┐ dispatch┌──────────┐
│ User │────────▶│ InputBar │──────▶│ ws-server│────────▶│ Gateway  │
└──────┘         └──────────┘       └──────────┘         └────┬─────┘
                                                               │
                              10+ Hook callbacks               │ Agent LLM
                                    ┌──────────────────────────┘
                                    ▼
┌──────────┐ GameEvent  ┌──────────────────┐ AgentEvent  ┌──────────────┐
│MainScene │◀───────────│ DirectorBridge   │◀────────────│    hook-     │
│          │ (65 types) │  Phase SM        │ (26+ types) │  translator  │
└────┬─────┘            └──────────────────┘             └──────────────┘
     │                          ▲
     │  GameAction (14 types)   │  workflow_phase_complete
     │  workstation_released    │  npc_move_completed
     └──────────────────────────┘
```

### Phase State Machine

```
         sub_agent          3s collect         phase              phase
         .started            window           complete            complete
┌──────┐ ─────────▶ ┌───────────┐ ────▶ ┌──────────┐ ────▶ ┌──────────────┐
│ idle │            │ summoning │       │assigning │       │going_to_office│
└──┬───┘            └───────────┘       └──────────┘       └──────┬───────┘
   ▲                                                               │
   │ phase                                                phase    │
   │ complete                                            complete  │
   │                                                               ▼
┌──┴───────┐ phase   ┌────────────┐  project   ┌─────────────────────┐
│returning │◀─ ── ── │ publishing │◀─ ── ── ── │       working       │
└──────────┘complete └────────────┘  _complete │                     │
                                               │  late sub_agent     │
                                               │  → assignLateArrival│
                                               └─────────────────────┘
```

### Citizen Independent Chat

```
┌──────┐  citizen   ┌──────────┐  WS    ┌──────────────┐  reply  ┌───────────┐
│ User │──  chat  ─▶│ InputBar │───────▶│ citizen-chat-│────────▶│Independent│
└──────┘            └──────────┘        │    router    │         │  Agent    │
   ▲                                    └──────────────┘         └─────┬─────┘
   │                                                                   │
   │  pushCitizenMessages                            Agent response    │
   └───────────────────────────────────────────────────────────────────┘
```

### NPC Implicit Behavior

```
┌────────────┐  implicit_chat   ┌──────────┐  HTTP  ┌───────────────┐
│ AgentBrain │──  _request   ──▶│ ws-server│───────▶│   llm-proxy   │
│ L1/L2/L3   │                  └──────────┘        │  2 parallel   │
└─────┬──────┘                       │              │  anthropic /  │
      ▲                              │              │  openai       │
      │  implicit_chat               │              └───────┬───────┘
      │  _response                   │                      │
      └──────────────────────────────┘◀─────────────────────┘
```

## Type Contracts

| Type | Definition | Variants | Consumers |
|------|-----------|----------|-----------|
| `AgentEvent` | `src/contracts/events.ts` | 22+ | DirectorBridge, hook-translator, ws-server |
| `GameEvent` | `town-frontend/src/data/GameProtocol.ts` | 65 | Bridge emit, EventDispatcher, MainScene |
| `GameAction` | same as above | 14 | MainScene → DataSource → Bridge |
| `PublishedCitizenConfig` | `town-frontend/src/data/CitizenWorkshopConfig.ts` | — | SceneBootstrap, editor-serve, CitizenWorkshop |

## Sub-Module AGENTS.md

| Module | Path | When to Read |
|--------|------|-------------|
| Plugin Layer | [src/plugin/AGENTS.md](src/plugin/AGENTS.md) | Modifying hooks / tools / plans / citizen agents / editor API |
| Bridge Layer | [src/bridge/AGENTS.md](src/bridge/AGENTS.md) | Modifying Phase state machine / event translation / pathfinding / citizen spawn |
| Town Frontend | [town-frontend/AGENTS.md](town-frontend/AGENTS.md) | Modifying any frontend UI / styles / text — **brand colors & i18n rules** |
| Frontend Game | [town-frontend/src/game/AGENTS.md](town-frontend/src/game/AGENTS.md) | Modifying scenes / workflow / mini-game / VFX / weather |

## Core Architecture Constraints

1. **Bridge emits high-level intents, not micro-ops** — Bridge only emits `workflow_*` / `npc_work_done` intent events; detailed animation choreography lives in `workflow/`
2. **NPC state machine driven** — Animations driven via `NPC.transitionTo(state)`, never call `playAnim()` directly
3. **Deferred workstation release** — Bridge does not release workstations immediately; waits for the frontend NPC to physically leave, confirmed via `workstation_released` callback
4. **Phase advances on frontend feedback** — Frontend sends `workflow_phase_complete` after finishing animations; only then does Bridge advance the state machine
5. **Citizen dual-track** — Citizens can be sub-agents (during work) or independent agents (daily chat, managed via `citizen-agent-manager`)

## Testing

```bash
# Plugin + bridge layer tests
npm test

# Frontend tests
cd town-frontend && npx vitest run
```

Test distribution:
- `src/bridge/__tests__/` — EventTranslator, RouteManager, ActivityStream, CitizenManager, NpcEventQueue
- `src/plugin/__tests__/` — hook-translator
- `town-frontend/src/game/__tests__/` — EventDispatcher, DialogManager, SceneSwitcher, GameClock, ModeManager, SceneBootstrap
- `town-frontend/src/data/__tests__/` — publishedToTownView

## Common Change Guide

| What to Do | Where to Start |
|-----------|---------------|
| Add Hook → AgentEvent mapping | `src/plugin/hook-translator.ts` |
| Add new AI tool | `src/plugin/tools.ts` |
| Modify multi-agent plan orchestration | `src/plugin/plan-manager.ts` |
| Modify editor backend API | `src/plugin/editor-serve.ts` |
| Modify citizen agent management | `src/plugin/citizen-agent-manager.ts` |
| Add AgentEvent → GameEvent mapping | `src/bridge/EventTranslator.ts` |
| Modify Phase state machine | `src/bridge/DirectorBridge.ts` |
| Modify citizen spawn animation | `src/bridge/CitizenManager.ts` |
| Modify NPC implicit behavior scenes | `src/bridge/implicit-chat.ts` |
| Modify workflow choreography | `town-frontend/src/game/workflow/Choreographer.ts` → corresponding Orchestrator |
| Modify NPC post-completion departure | `town-frontend/src/game/workflow/WorkflowHandler.ts` `handleNpcWorkDone()` |
| Add frontend GameEvent handler | `town-frontend/src/game/EventDispatcher.ts` |
| Modify NPC daily behavior | `town-frontend/src/game/DailyScheduler.ts` + `npc/DailyBehavior.ts` |
| Modify NPC animation / state machine | `town-frontend/src/npc/NPC.ts` — driven by `transitionTo()` |
| Modify casual social encounters | `town-frontend/src/npc/CasualEncounter.ts` + `DialogueScripts.ts` |
| Modify mini-game | `town-frontend/src/game/minigame/BanweiGame.ts` |
| Modify weather effects | `WeatherSystem.ts` (state machine) + `WeatherEffects.ts` (visual) + `AmbientSoundManager.ts` (audio) |
| Modify 3D scenes | `town-frontend/src/game/scene/TownBuilder.ts` |
| Modify editor | `town-frontend/src/editor/` |
| Modify citizen workshop | `town-frontend/src/editor/citizen/CitizenWorkshop.ts` |
| Modify character model system | `town-frontend/src/data/CharacterModelRegistry.ts` |
| Add soul file | `town-souls/*.md` |
| Modify audio | `town-frontend/src/audio/` corresponding file |

---
> Source: [Agentshire/Agentshire](https://github.com/Agentshire/Agentshire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
