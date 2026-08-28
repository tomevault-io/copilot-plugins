## viableos

> ViableOS — "The operating system for viable AI agent organizations."

# CLAUDE.md — ViableOS

## Projekt

ViableOS — "The operating system for viable AI agent organizations."
Design-Tool das Organisationen auf Basis des Viable System Model (VSM) strukturiert und Agent-Konfigurationen generiert.

## Tech Stack

- **Backend**: Python 3.10+, FastAPI, LiteLLM, PyMuPDF
- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS 4, Zustand
- **Kein DB** — alles in-memory (Sessions, FileStore). Ephemeral by design.

## Architektur

```
src/viableos/
├── api/
│   ├── chat_routes.py    # Chat SSE streaming + file upload
│   └── routes.py         # Wizard/config API endpoints
├── chat/
│   ├── engine.py         # LiteLLM streaming, multimodal messages
│   ├── session.py        # In-memory session + message store
│   ├── files.py          # FileStore: PDF/image/text upload processing
│   └── system_prompt.py  # 650-line VSM expert prompt (NEVER expose theory to user)
├── checker.py            # Viability checks (S1-S5) + community warnings
├── assessment_transformer.py  # Assessment JSON → viable_system config
├── budget.py             # Token budget calculation
├── coordination.py       # Auto-generated coordination rules
├── generator.py          # OpenClaw package generator
└── langgraph_generator.py

frontend/src/
├── components/chat/      # ChatWindow, MessageBubble, ProviderSelector, AssessmentPreview
├── components/wizard/    # 6-step wizard (Template→Units→Budget→Identity→HITL→Review)
├── store/                # Zustand stores (useChatStore, useConfigStore)
├── api/client.ts         # Typed API client
└── types/index.ts        # All TypeScript interfaces
```

## Chat Flow

1. User chattet mit VSM-Experte (LiteLLM streaming via SSE)
2. Kann PDFs, Screenshots, Dateien hochladen (drag&drop, paste, paperclip)
3. LLM führt 4-Phasen Assessment-Interview
4. "Finalize Assessment" extrahiert JSON aus letzter Assistant-Message
5. "Use in Wizard" transformiert → öffnet Config-Wizard
6. Wizard Step 6 (ReviewStep) ruft `/api/check` für Viability-Report

## SSE Streaming

**Wichtig**: Chunks werden JSON-encoded gesendet (`json.dumps(chunk)`), damit Newlines in LLM-Output das SSE-Format nicht brechen. Frontend parst mit `JSON.parse()`.

## Bekannte Patterns

- API Keys nur in-memory (nie persistiert, kein localStorage)
- CSS Variables statt hardcoded colors (`var(--color-primary)` etc.)
- Provider-Abstraktion über LiteLLM (Anthropic, OpenAI, Google, DeepSeek, xAI, Ollama)
- `system_prompt.py` enthält tiefes VSM-Wissen, aber der LLM darf NIEMALS akademische Begriffe verwenden

## Testing

### Test-Befehle
```bash
# TypeScript prüfen
cd frontend && npx tsc --noEmit

# Linting
cd frontend && npm run lint

# Unit Tests (Vitest)
cd frontend && vitest run

# E2E Tests (Playwright – startet Dev-Server automatisch)
cd frontend && npx playwright test

# Backend Tests
pytest tests/ -v
```

### Test-Reihenfolge bei Code-Änderungen
1. **TypeScript Check**: `npx tsc --noEmit` → Muss ohne Fehler durchlaufen
2. **Unit Tests**: `vitest run` → Alle grün?
3. **E2E Tests**: `npx playwright test` → Alle grün?
4. **Backend Tests**: `pytest tests/ -v` → Alle grün?
5. Erst wenn alles grün ist: committen

### Wenn Tests fehlschlagen
- Playwright Screenshots: `frontend/tests/e2e/screenshots/`
- Playwright HTML-Report: `cd frontend && npx playwright show-report`
- Console-Errors werden direkt im Test-Output angezeigt
- Fix den Fehler und lauf die Tests erneut bevor du weiter machst

## Gotchas

- `checker.py`: success_criteria priorities können `int` oder `string` sein — immer mit `str()` casten
- `assessment_transformer.py`: Gibt `{"viable_system": {...}}` zurück, nicht nur das viable_system
- FileStore ist global singleton (`file_store`) — kein Cleanup implementiert
- `max_tokens=4096` in engine.py — bei langen Assessment-JSONs ggf. zu wenig

---
> Source: [philipp-lm/ViableOS](https://github.com/philipp-lm/ViableOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
