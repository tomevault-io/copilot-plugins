## geschichtenerzaehler

> KI-basierte interaktive Kinderlern- und Abenteuer-App. Erzeugt dynamisch Geschichten mit adaptiven Rätseln für Grundschulkinder (6-10 Jahre).

# Geschichtenerzähler — "Zauberwald"

KI-basierte interaktive Kinderlern- und Abenteuer-App. Erzeugt dynamisch Geschichten mit adaptiven Rätseln für Grundschulkinder (6-10 Jahre).

## Tech-Stack
- **Frontend:** React 19 (TypeScript), Vite 6, react-router-dom 7
- **Backend:** Node.js (TypeScript), Express 4, tsx
- **LLM:** Anthropic SDK, OpenAI SDK (Provider-Abstraktion)
- **Validation:** Zod
- **Persistenz:** JSON-Files (MVP), später DB
- **Paketmanager:** npm Workspaces

## Architektur

### Verzeichnisse
- `client/` — React Frontend (Vite, Port 5173, Proxy → 3001)
- `server/` — Express Backend / API (Port 3001)
- `shared/` — Geteilte TypeScript-Typen, Enums, Zod-Schemas, Konstanten
- `content/` — Statische Inhalte (Setting-Bibel, Prompts, Story-Arcs, Puzzle-Templates)
- `docs/` — Produktkonzept, Architektur, Datenmodell, Roadmap, Safety-Rules
- `tests/` — Tests (server/, shared/)

### Story Engine — 9 Schichten
1. **World Lore Layer** — Setting-Bibel laden, per Tag-System filtern
2. **Canon Rules Layer** — Stil/Sicherheit/Charakter-Regeln (jedem LLM-Call vorangestellt)
3. **Player State Layer** — Inventar, Gefährten, Mastery als Prompt-Block
4. **Recent Memory Layer** — Sliding Window (letzte 5 Szenen-Summaries)
5. **Scene Planner** — LLM (Sonnet): Ort, Rätseltyp, Schwierigkeit → JSON-Skeleton
6. **Scene Writer** — LLM (Sonnet): Kindgerechte Szene ausformulieren
7. **Response Judge** — LLM (Haiku): Kinderantworten flexibel bewerten
8. **State Updater** — Logisch: StateChanges verarbeiten (Mastery/Inventar/Trust)
9. **Summarizer** — LLM (Haiku): Kapitel-Zusammenfassungen

### LLM Routing
| Task | Modell | Provider |
|------|--------|----------|
| Scene Planner | claude-sonnet-4-6 | Anthropic |
| Scene Writer | claude-sonnet-4-6 | Anthropic |
| Response Judge | claude-haiku-4-5 | Anthropic |
| Summarizer | claude-haiku-4-5 | Anthropic |

### Server-Struktur
- `routes/` — REST-Endpunkte (player, story, puzzle, inventory, companion, guardian)
- `controllers/` — Request-Handler
- `services/story-engine/` — Die 9 Schichten (Kern)
- `services/puzzle-engine/` — Generator, Evaluator, Difficulty, Mastery
- `services/llm/` — Provider Interface, Anthropic/OpenAI Adapter, Router, Prompt Builder
- `services/safety/` — Content-Filter, Rate-Limiter
- `data/` — JSON-Store Abstraktion, Repositories

### Client-Struktur
- `pages/` — Home, Story, Map, Inventory, Guardian
- `components/` — ui, story, puzzle, companion, inventory, map, player, guardian
- `hooks/` — useStory, usePuzzle, usePlayer, useAudio, useApi
- `services/api.ts` — API-Client
- `styles/` — Kindgerechtes Waldthema (warme Farben, große Typografie)

## Code-Regeln
- Keine Secrets im Code
- TypeScript strict mode
- Saubere Fehlermeldungen
- Code kommentieren wo nicht selbsterklaerend
- Funktionale React-Komponenten mit Hooks
- Safety Preamble bei jedem LLM-Call
- Definition vs. Instanz trennen (statisch in content/ vs. dynamisch in data/)

## Referenz-Dateien
- Projektkontext: `CLAUDE.md` (diese Datei)
- Session-Log: `STATUS.md`
- Setting-Bibel: `docs/Zauberwald – Setting-Bibel (1).pdf`
- Produktkonzept: `docs/product-concept.md`
- Architektur: `docs/system-architecture.md`
- Datenmodell: `docs/data-model.md`
- Story Engine: `docs/story-engine.md`
- Safety: `docs/safety-rules.md`
- Roadmap: `docs/roadmap.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mark296688) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
