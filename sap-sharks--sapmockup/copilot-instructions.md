## sapmockup

> SpecForge is a web-based drag-and-drop SAP screen mockup tool. It lets SAP consultants

# SpecForge — SAP Mockup Studio
## Master Instructions for Claude Code Agent

---

## What this project is

SpecForge is a web-based drag-and-drop SAP screen mockup tool. It lets SAP consultants
prototype any SAP screen (Classic GUI, Fiori 3, or S/4HANA Horizon) in minutes —
without Figma, without SAP access, without a developer.

It fills the gap left by SAP Build (sunset 2020), which had 1,000+ prototypes/week
at peak. No replacement exists for custom (non-Fiori Elements) SAP screen prototyping.

---

## Tech stack — do not deviate from this

| Layer        | Technology                        | Reason                                      |
|--------------|-----------------------------------|---------------------------------------------|
| Frontend     | React 18 + TypeScript             | Component model suits drag-drop canvas      |
| Build        | Vite                              | Fast HMR, simple config                     |
| UI (app UI)  | Tailwind CSS                      | Our own tool's UI                           |
| SAP Fiori    | UI5 Web Components v2.20.0        | Real SAP components, Apache 2.0 licensed    |
| State        | Zustand                           | Simple, no boilerplate                      |
| Drag & drop  | dnd-kit                           | Best React DnD, accessible                  |
| Routing      | React Router v6                   | SPA routing                                 |
| Backend      | FastAPI (Python)                  | AI endpoints, project save/load             |
| Database     | SQLite (dev) / PostgreSQL (prod)  | Project persistence                         |
| AI           | Anthropic Claude API (claude-sonnet-4-5) | Interview engine, slot filling      |
| Export       | html2canvas + jsPDF               | PNG/PDF export of mockups                   |

---

## Project structure

```
specforge/
├── CLAUDE.md                  ← this file (read first, always)
├── AGENTS.md                  ← agent roles and task ownership
├── PLAN.md                    ← phased build plan with tasks
├── docs/
│   ├── SAP_UI_VERSIONS.md     ← SAP GUI vs Fiori 3 vs Horizon specs
│   ├── COMPONENTS.md          ← every SAP component, its slots, props
│   ├── FLOORPLANS.md          ← SAP floorplan layouts and rules
│   ├── SLOT_SCHEMA.md         ← JSON schema for AI slot filling
│   └── API.md                 ← backend API contract
├── src/
│   ├── components/
│   │   ├── canvas/            ← drag-drop canvas and grid
│   │   ├── palette/           ← component sidebar palette
│   │   ├── properties/        ← right-panel property editor
│   │   ├── sap-classic/       ← Classic SAP GUI component renderers
│   │   ├── sap-fiori3/        ← Fiori 3 / Quartz component renderers
│   │   ├── sap-horizon/       ← Horizon / S/4HANA component renderers
│   │   └── shared/            ← shared app UI components
│   ├── stores/
│   │   ├── canvasStore.ts     ← canvas state (components, positions)
│   │   ├── projectStore.ts    ← project metadata, save/load
│   │   └── uiStore.ts         ← app UI state (active panel, mode)
│   ├── agents/
│   │   ├── interviewAgent.ts  ← AI interview conversation manager
│   │   ├── slotFillerAgent.ts ← maps FS JSON → component slot values
│   │   └── specAgent.ts       ← generates FS/TS documents from interview
│   ├── lib/
│   │   ├── sapTables.ts       ← SAP table/field knowledge base
│   │   ├── floorplans.ts      ← floorplan template definitions
│   │   ├── componentRegistry.ts ← all draggable components + metadata
│   │   └── export.ts          ← PNG/PDF/shareable link export
│   └── pages/
│       ├── Home.tsx            ← project list
│       ├── Builder.tsx         ← main canvas page
│       ├── Interview.tsx       ← AI interview flow
│       └── Preview.tsx         ← fullscreen shareable preview
└── backend/
    ├── main.py                ← FastAPI app entry
    ├── routers/
    │   ├── projects.py        ← CRUD for projects
    │   ├── interview.py       ← Claude API interview endpoint
    │   └── export.py          ← server-side export
    ├── models.py              ← SQLAlchemy models
    └── claude_client.py       ← Anthropic SDK wrapper
```

---

## Core concepts — understand these before writing any code

### 1. UI Version
Every element on the canvas belongs to one of three SAP UI versions:
- `classic` — SAP GUI (ECC, old Windows look, #003399 blue, monospace fonts)
- `fiori3` — Fiori 3 / Quartz Light (flat, white, SAP blue #0070F2)
- `horizon` — Morning Horizon / S/4HANA current (rounded, #0064D9, softer)

A project has ONE ui_version. All components on canvas must match it.
Never mix ui_versions on one canvas.

### 2. Floorplan
A floorplan is a layout shell — the structural skeleton of the screen.
Components are placed INSIDE floorplan regions (header, content, footer, sidebar).
Available floorplans per version:

**Classic:** SelectionScreen, ALVGrid, DialogTransaction, DocumentDisplay
**Fiori 3:** ListReport, ObjectPage, OverviewPage, Worklist, AnalyticalList
**Horizon:** Same as Fiori 3 + Flexible Column Layout

### 3. Component
A component is a draggable SAP UI element. Each has:
- `type` — e.g. "input", "checkbox", "table", "button"
- `version` — which ui_version it belongs to
- `slots` — named content slots the AI fills (label, placeholder, options, etc.)
- `props` — visual/behavioural props (required, disabled, width, etc.)

### 4. Slot filling
The AI interview produces a structured JSON (the "spec"). The slotFillerAgent
reads the spec and maps it to component slot values. The canvas renders the
result. Humans can then override any slot manually in the properties panel.

### 5. Canvas state shape
```typescript
interface CanvasComponent {
  id: string
  type: string           // "input" | "button" | "checkbox" | "table" | ...
  version: UIVersion     // "classic" | "fiori3" | "horizon"
  regionId: string       // which floorplan region it belongs to
  position: { x: number; y: number }
  size: { w: number; h: number }
  slots: Record<string, string>
  props: Record<string, unknown>
  zIndex: number
}

interface CanvasState {
  projectId: string
  uiVersion: UIVersion
  floorplan: string
  components: CanvasComponent[]
  selectedId: string | null
}
```

---

## Coding rules — enforce these always

1. **TypeScript strict mode** — no `any`, no `ts-ignore`
2. **No inline styles** — use Tailwind classes for app UI, CSS modules for SAP renderers
3. **Component files max 200 lines** — split if longer
4. **Every SAP renderer is isolated** — classic/ fiori3/ horizon/ folders never import from each other
5. **All AI calls go through `claude_client.py`** — never call Anthropic API directly from frontend
6. **Canvas state is the single source of truth** — never duplicate state in component local state
7. **Slot values are always strings** — type coerce at render time, not in state
8. **Export never modifies state** — export functions are pure, read-only
9. **All backend routes return `{ data, error }` envelope** — consistent API shape
10. **Write tests for agents** — interviewAgent and slotFillerAgent must have unit tests

---

## Environment variables required

```bash
# .env (never commit)
ANTHROPIC_API_KEY=sk-ant-...
DATABASE_URL=sqlite:///./specforge.db    # or postgres URL for prod
VITE_API_BASE=http://localhost:8000
ALLOWED_ORIGINS=http://localhost:5173
```

---

## Commands

```bash
# Frontend
npm install
npm run dev          # starts Vite on :5173
npm run build
npm run test

# Backend
pip install -r requirements.txt
uvicorn backend.main:app --reload --port 8000

# Both together (dev)
npm run dev:all      # uses concurrently
```

---

## What NOT to build (out of scope for Phase 1)

- User authentication / multi-tenant (Phase 2)
- Real-time collaboration (Phase 3)
- ABAP code generation (Phase 2)
- Full FS/TS document export (Phase 2 — interview agent only in Phase 1)
- Mobile / responsive canvas (desktop only for now)
- SAP system connectivity / live data (Phase 3)

---

## When you are unsure

1. Check `docs/COMPONENTS.md` for component spec questions
2. Check `docs/FLOORPLANS.md` for layout questions
3. Check `docs/SLOT_SCHEMA.md` for AI/slot questions
4. Check `AGENTS.md` for which agent owns which responsibility
5. Ask — do not guess on SAP-specific behaviour

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SAP-SHARKS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
