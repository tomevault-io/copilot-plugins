## sessionscribe

> 1. Mission: A Windows desktop app for therapy sessions with **live transcription (faster-whisper), PHI redaction (spaCy+regex), and DAP note generation (OpenAI)**.


# SessionScribe Windsurf Project Rules

## 0) Mission & Non-Goals
1. Mission: A Windows desktop app for therapy sessions with **live transcription (faster-whisper), PHI redaction (spaCy+regex), and DAP note generation (OpenAI)**.
2. Non-Goals: No cloud PHI; no logging PHI; no speculative refactors beyond tasks. (Source: StatusPlan_09202025.md) 

## 1) Tech & Architecture Truths (authoritative)
1. Desktop: **Electron 31 + React 18 + Vite** (`apps/desktop/electron`, `apps/desktop/renderer`).
2. Services (FastAPI, Python 3.11+):
   - ASR @ **7031**
   - Redaction @ **7032**
   - Insights Bridge @ **7033**
   - Note Builder @ **7034**
3. Audio: **WASAPI**, 48kHz stereo PCM; target **P95 latency ≤ 2.0s**.
4. Start commands:
   - `.\scripts\dev.ps1` (all)
   - Manual:
     - `.\.venv\Scripts\python.exe -m uvicorn services.asr.app:app --host 127.0.0.1 --port 7031`
     - `pnpm -C apps/desktop/renderer dev --port 3001`
     - `pnpm -C apps/desktop/electron dev`
5. Tests:
   - `pytest`
   - `pnpm -C apps/desktop/renderer test`
   - `pnpm -C apps/desktop/renderer e2e`
(Authoritative details: StatusPlan_09202025.md)

## 2) Safety & Privacy Guardrails (must-follow)
1. **Never** emit PHI in logs, chat, or code comments. Replace with neutral labels (e.g., `ClientA`).
2. **Do not** call external APIs with unredacted content. Redaction step is mandatory before any network use.
3. **No edits** to logging config that could expose PHI. If logging changes are needed, propose diff + threat model first.

## 3) Engineering Workflow (how Cascade should act)
1. **Ask 1–2 clarifying Qs** before code changes if task is underspecified; otherwise proceed.
2. **Task-driven edits only.** Provide a **mini-plan + file-level diff** preview before writing.
3. After changes, produce a **Verification Checklist** that includes:
   - Run cmds used
   - What you observed (latency, crash repro)
   - Tests added/updated
4. If touching ASR/WS pipelines, include a **failure-mode note** (timeouts, reconnection, null-guards).

## 4) File & Folder Protections
1. **Do not edit**: `packages/schemas/*` without schema version bump + validation.
2. **Do not move/rename** Electron preload bridges without updating IPC contracts.
3. **Do not add** new dependencies that send telemetry or could leak PHI.
4. If changing ports or WebSocket paths, update **renderer env + Electron IPC** in the same PR.

## 5) Coding & UX Conventions
1. **TypeScript**: strict, no `any` without justification; add guards for `undefined` in UI state.
2. **React**: no alerts; prefer toasts + `ErrorBoundary`; maintain <1s UI update latency.
3. **Python/FastAPI**: pydantic validation at boundaries; explicit timeouts/retries; circuit-breaker around service calls.
4. **Accessibility**: aria-labels on interactive controls; keyboard shortcuts documented.

## 6) Debug & Test Rules (current priorities)
1. For the **LiveTranscriber.tsx undefined 'transcription' crash**:
   - Add null guards + suspense fallback.
   - Validate WS message schema + reconnect strategy.
   - Provide minimal repro + unit test.
2. Expand coverage for audio pipeline and E2E session flow.
3. Maintain **P95 ≤ 2.0s** under load; include bench notes.

## 7) Output Formats from Cascade
- When proposing changes, return:
  1) **Plan** (bulleted),  
  2) **Diffs** (per file snippets),  
  3) **Run & Test cmds**,  
  4) **Verification Checklist**,  
  5) **Roll-back notes**.

*(Source of truths & ports/tests/targets: StatusPlan_09202025.md)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masterzoidberg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
