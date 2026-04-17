## fs-node-project

> **FS Node Project** is a visual, node-based editor for building and executing AI pipelines. It allows users to create complex generative AI workflows for image, video, and audio processing by connecting functional nodes on a canvas.

# GEMINI.md - FS Node Project

## Project Overview
**FS Node Project** is a visual, node-based editor for building and executing AI pipelines. It allows users to create complex generative AI workflows for image, video, and audio processing by connecting functional nodes on a canvas.

### Core Philosophy
- **Nodes as API Interfaces:** Every node is a visual representation of an AI API endpoint (Freepik, Anthropic, ElevenLabs, etc.).
- **Typed Connections:** Data flows through edges with strict type validation (image, video, audio, text, etc.), represented by colored handles.
- **Async Execution:** Heavy AI tasks follow a "Submit -> Poll -> Complete" pattern.

### Main Technologies
- **Frontend:** React 19 (StrictMode), Vite 8, @xyflow/react (React Flow), Vitest.
- **Backend:** Node.js (Express 5), Modular Routes & Services, Multer.
- **Persistence:** Firebase Firestore (with local state fallback).
- **AI Integrations:** Freepik (Generative AI), Anthropic (Claude Vision), ElevenLabs (Audio), Kling/Runway/MiniMax (Video).
- **Deployment:** Vercel.

---

## Building and Running

### Global Build
Prepare the project for production (builds frontend, installs api deps, copies dist to public):
```bash
npm run build
```

### Frontend Development
```bash
cd frontend
npm run dev      # Start Vite dev server (port 5173/5175)
npm run lint     # Run ESLint
npm run test     # Run Vitest suite
```

### Backend Development
```bash
cd api
npm start        # Start Express server (port 3001)
```

### Vision & Regression Testing
- **Visual Regression:** `cd frontend && node visual-compare.mjs` (Playwright-based).
- **Vision Tests:** Run various `test_*.py` scripts in the root using Gemini API to validate UI components.

---

## Development Conventions

### Node System
- **Location:** All nodes are in `frontend/src/nodes/`.
- **Infrastructure:** Use `frontend/src/nodes/shared.js` for design tokens, common UI components (Pills, Sliders), and hooks (`useNodeConnections`, `useNodeExecution`).
- **Registration:** New nodes must be registered in `frontend/src/App.jsx` (`nodeTypes` and `NODE_MENU`).

### Connection Colors & Types
Defined in `frontend/src/utils/handleTypes.js`:
- **Image:** Pink (`#ec4899`)
- **Video:** Teal (`#14b8a6`)
- **Text:** Orange (`#f97316`)
- **Audio:** Purple (`#a855f7`)
- **Aspect Ratio:** Amber (`#f59e0b`)
- **Resolution:** Green (`#22c55e`)

### Backend API
- **Modular Routes:** Routes are separated into `api/routes/` (images, videos, audio, etc.).
- **Services:** External API logic resides in `api/services/`.
- **Environment:** Requires `FREEPIK_API_KEY`, `ANTHROPIC_API_KEY`, and `ELEVENLABS_API_KEY` in `api/.env`.

### Firebase Integration
- Managed via `frontend/src/hooks/useFirebaseWorkflows.ts`.
- Automatically falls back to local browser state if Firebase credentials are not provided in the environment.

### Patching & Automation
The project contains multiple `patch_*.js/py` scripts in the root for automated codebase migrations, styling updates, and bulk refactors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/felixseeger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
