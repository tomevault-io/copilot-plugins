## medkit-app

> Browser-based ER + polyclinic clinical training simulator. Doctor-POV game: new patients arrive at triage, you diagnose, order tests, treat, disposition. Voice conversations with the patient run real-time over LiveKit: Deepgram Nova-3 (STT) + Claude Haiku 4.5 (dialog) + Cartesia Sonic-2 (TTS). Polyclinic is a second flow — one outpatient at a time, tests resolve instantly.

# medkit — Claude Code project notes

Browser-based ER + polyclinic clinical training simulator. Doctor-POV game: new patients arrive at triage, you diagnose, order tests, treat, disposition. Voice conversations with the patient run real-time over LiveKit: Deepgram Nova-3 (STT) + Claude Haiku 4.5 (dialog) + Cartesia Sonic-2 (TTS). Polyclinic is a second flow — one outpatient at a time, tests resolve instantly.

## Tech stack

- **Frontend:** React 18 + TypeScript + Vite, Three.js via `@react-three/fiber` and `@react-three/drei`.
- **Voice (transport):** LiveKit Cloud (WebRTC). Browser publishes mic, subscribes to remote audio. `livekit-client` in the browser; the worker lives in `backend/voice_agent.py`.
- **Voice (backend worker):** `livekit-agents` Python framework with `deepgram` STT, `anthropic` LLM (Haiku 4.5), `cartesia` TTS, `silero` VAD. Runs in its own venv (`backend/.venv-voice`).
- **Backend HTTP:** FastAPI at `http://127.0.0.1:8787` — Managed Agents proxy + `/voice/token` mint (creates the LiveKit room with patient persona metadata, returns a JWT for the browser).
- **LLM (attending grading):** Anthropic SDK server-side. Patient persona Haiku 4.5 lives inside the LiveKit agent; the medkit-attending Managed Agent (Opus 4.7) lives in `backend/server.py`.
- **State:** single `Store` class with `useSyncExternalStore` (see `src/game/store.ts`). No Redux/Zustand — don't add one.

## Key files (what to read first)

- `src/game/store.ts` — single source of truth for all game state (ER beds, polyclinic, notifications, archive).
- `src/game/types.ts` — `PatientCase`, `ActivePatient`, `GameState`.
- `src/data/patients.ts` / `polyclinicPatients.ts` / `tests.ts` / `treatments.ts` / `medications.ts` — pure data files.
- `src/components/three/Polyclinic.tsx` — polyclinic 3D scene, currently active area of work.
- `src/voice/conversation.ts` — `Conversation` class, LiveKit-backed real-time session (mic, remote audio, transcription events, lip-sync analyser tap).
- `src/voice/conversationStore.ts` — per-bed conversation cache; T-toggle and patient-leaves dispose explicitly.
- `src/voice/patientPersona.ts` — system prompt builder. Adult vs pediatric (parent speaks for child).
- `src/voice/claude.ts` — Anthropic SDK wrapper with prompt caching, used only by the text-chat path now.
- `src/agents/managedAgent.ts` / `customTools.ts` / `eventStreamRenderer.tsx` — Claude Managed Agents integration (the attending physician). See `.claude/skills/medkit-managed-agent-setup.md`.
- `backend/server.py` — FastAPI: Managed Agents proxy under `/agent/*` + `/voice/token`.
- `backend/voice_agent.py` — LiveKit Agents worker. Reads room metadata for persona + voice ID and wires Deepgram → Haiku → Cartesia.
- `spec.md` — hackathon submission plan, canonical source for Managed Agents scope.

## Commands

- `npm run dev` — Vite dev server.
- `npm run build` — tsc + vite build.
- `npm run preview` — preview production build.
- `npm run verify` — deterministic invariant checks on `src/data/*` (see `.claude/skills/medkit-verify-simulation.md`). Run after every data/type/store edit.
- Backend (FastAPI): `backend/.venv/Scripts/python.exe backend/server.py` — listens on 8787, hosts Managed Agents proxy + `/voice/token`.
- Voice worker: `backend/.venv-voice/Scripts/python.exe backend/voice_agent.py dev` — separate process, registers with LiveKit Cloud and dispatches into rooms created by `/voice/token`. Both processes must be up for voice to work.

### Running node-binary wrappers on this machine

Group policy on BrynQ dev machines blocks `.exe` wrappers under `node_modules/.bin/` — this hits `tsx.exe`, `vite.exe`, `tsc.exe`, `npx`, and anything else that gets compiled to a shim executable. Symptom: `This program is blocked by group policy`.

Workarounds baked into the repo:
- `npm run dev` → `node node_modules/vite/bin/vite.js`
- `npm run build` → `node node_modules/typescript/bin/tsc && node node_modules/vite/bin/vite.js build`
- `npm run verify` → `node scripts/verify/run-all.ts` (Node 22+ runs `.ts` natively via type-stripping)

When adding a new script that would normally invoke a binary wrapper, use the same `node node_modules/<pkg>/bin/<entry>.js` pattern. Do not add `tsx` as a devDependency.

## How to work in this repo

- **Keep changes minimal.** Bug fix = bug fix. Don't refactor adjacent code "while you're there."
- **Data files are data.** If you're adding a new case/test/medication, edit the data file — don't plumb new shape through the store unless the game mechanic genuinely changed.
- **Three.js scene edits:** the polyclinic and ER rooms have fixed floor/wall dimensions. New meshes must respect the floor plane and not overlap existing furniture — verify by running the dev server and rotating the camera, not by eyeballing numbers.
- **No new state libraries.** The `Store` class handles everything. If you need derived state, compute it in a selector or in the component.
- **Voice runs over the network now.** Browser doesn't load STT/TTS models — Deepgram + Cartesia are upstream services reached via the LiveKit room. There's nothing to preload on the frontend.
- **Prompt caching is on** in `src/voice/claude.ts`. When you add a new Claude call, set `cache_control: { type: 'ephemeral' }` on the system prompt.

## Model routing

| Call | Model | Why |
|---|---|---|
| Patient voice persona (in the LiveKit agent) | Haiku 4.5 | Fast, cheap, good enough for in-character reply |
| `medkit-attending` Managed Agent (clinical grading) | **Opus 4.7** | Clinical reasoning, precision matters |
| Demo video narration generation | Opus 4.7 | One-off, polish matters |

When you add a new Claude-backed feature, decide which bucket it falls in.

## API keys

All keys server-side only, in `backend/.env.local`. The browser never sees them.

- `ANTHROPIC_API_KEY` — used by the FastAPI server (Managed Agents + `/agent/patient/stream` for the text-chat path) AND by the LiveKit voice worker (Haiku patient persona).
- `LIVEKIT_URL` / `LIVEKIT_API_KEY` / `LIVEKIT_API_SECRET` — used by FastAPI to mint room JWTs and by the worker to register.
- `DEEPGRAM_API_KEY` — voice worker (streaming STT).
- `CARTESIA_API_KEY` — voice worker (streaming TTS).

Vite proxies `/agent/*` and `/voice/*` to `127.0.0.1:8787` in dev. When you add a new Claude-backed feature, route it through the backend the same way — never reintroduce a `VITE_*` Anthropic key.

## Out of scope

- Multi-agent handoffs (Anthropic's feature is still in research preview).
- Outcomes / self-verifying rubrics (wait-list only).
- A persistent user account system — the simulator is single-player, single-shift.
- Any medical claim of clinical accuracy. Cases are plausible but synthetic.

## Don't

- Don't create specialist sub-agents (`triage-expert`, `pharmacology-expert`, …). Use skills in `.claude/skills/` and let Claude compose them.
- Don't write long rigid bullet lists of rules here — Opus 4.7 follows them literally and over-triggers. Encode hard rules as hooks or lint instead.
- Don't commit `.env.local`, voice sample clips over 1 MB, or anything under `node_modules/`, `backend/.venv/`, `dist/`.

---
> Source: [bedriyan/medkit-app](https://github.com/bedriyan/medkit-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
