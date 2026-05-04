## digitaltwin

> This file documents the AI agent configuration for the HVAC Digital Twin project, designed for use with GitHub Copilot and compatible coding agents.

# AGENTS.md: Copilot Agent Configuration

This file documents the AI agent configuration for the HVAC Digital Twin project, designed for use with GitHub Copilot and compatible coding agents.

## Project Context

This is a full-stack web application (React + Node.js) that simulates a multi-floor office building's HVAC system. It uses **Foundry Local** for on-device AI inference to power an operations copilot.

## Workspace Structure

| Folder | Purpose |
|--------|---------|
| `frontend/` | React + Three.js web UI (Vite dev server on port 3000) |
| `backend/` | Node.js/Express API server (port 3001) + HVAC simulator |
| `twin/` | JSON-based digital twin state, schema, and baseline |
| `tests/` | Integration, validation, and E2E tests |
| `docs/` | Architecture diagrams, demo scripts, images |
| `assets/` | 3D model generation scripts |

## Technology Stack

- **Frontend:** React 18, Three.js (@react-three/fiber, @react-three/drei), Zustand, Vite
- **Backend:** Node.js 20+, Express, WebSocket (ws)
- **AI:** Foundry Local SDK (`foundry-local-sdk` npm package), on-device SLM inference
- **Testing:** Node.js built-in test runner, Playwright (screenshots)
- **Package format:** ESM (`"type": "module"` in package.json)

## Coding Conventions

- **Module system:** ES Modules (`import`/`export`). Do not use `require()`.
- **Style:** Vanilla CSS with CSS custom properties (no Tailwind, no CSS-in-JS).
- **State management:** Zustand store in `frontend/src/hooks/useTwinStore.js`.
- **API pattern:** REST endpoints under `/api/twin/*` and `/api/copilot/*`. WebSocket on `/ws`.
- **File naming:** Components use PascalCase (e.g., `BuildingScene.jsx`), services use kebab-case (e.g., `copilot-service.js`).
- **No TypeScript**: plain JavaScript with JSDoc for documentation.

## Key Files

| File | Description |
|------|-------------|
| `backend/src/index.js` | Express server, routes, WebSocket, Foundry Local initialisation |
| `backend/src/services/copilot-service.js` | Intent detection, grounded responses, action execution |
| `backend/src/services/foundry-local-service.js` | Foundry Local SDK lifecycle (download, load, chat) |
| `backend/src/simulator/hvac-simulator.js` | Deterministic HVAC physics simulation |
| `frontend/src/App.jsx` | Main layout with 3D viewer, panels, model status banner |
| `frontend/src/hooks/useTwinStore.js` | Zustand store: twin state, WebSocket, copilot, model status |
| `twin/twin.state.json` | Live twin state (mutable at runtime) |
| `twin/twin.baseline.json` | Baseline state used for resets |
| `twin/twin.schema.json` | JSON Schema for twin state validation |

## Foundry Local Integration

The backend uses the `foundry-local-sdk` npm package (not CLI commands or raw HTTP). Key patterns:

```javascript
import { FoundryLocalManager } from 'foundry-local-sdk';

const manager = FoundryLocalManager.create({ appName: 'hvac-digital-twin', logLevel: 'info' });
const model = await manager.catalog.getModel('phi-3.5-mini');
await model.download((progress) => console.log(`${progress}%`));
await model.load();
const chatClient = model.createChatClient();
const result = await chatClient.completeChat([{ role: 'user', content: '...' }]);
```

## Running the Project

```bash
# Backend
cd backend && npm install && node src/index.js

# Frontend (separate terminal)
cd frontend && npm install && npm run dev
```

## Testing

```bash
cd backend && npm test          # Unit tests
cd tests && npm test            # Integration tests
```

## Important Notes

- The twin state JSON files are read/written at runtime. Be careful with concurrent edits.
- Foundry Local model download can take several minutes on first run. The UI shows a progress banner.
- On Windows, install the SDK with `npm install --foreground-scripts --winml foundry-local-sdk`.
- The copilot fallback system provides built-in responses when the AI model is unavailable.

---
> Source: [leestott/DigitalTwin](https://github.com/leestott/DigitalTwin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
