## commcare-forge

> CommCare Forge is a desktop app (Electron) that generates CommCare mobile applications from natural language descriptions or uploaded documents. It uses Claude (Anthropic API) to power a conversational interface where users describe their app, then generates, validates, and exports a complete CommCare application ready for import into CommCare HQ.

# CommCare Forge — Agent Guidelines

## What This Project Does

CommCare Forge is a desktop app (Electron) that generates CommCare mobile applications from natural language descriptions or uploaded documents. It uses Claude (Anthropic API) to power a conversational interface where users describe their app, then generates, validates, and exports a complete CommCare application ready for import into CommCare HQ.

## Architecture

- **Desktop**: Electron (main + renderer process)
- **Frontend**: React + TypeScript + Tailwind CSS (renderer process)
- **Backend**: Node.js + TypeScript (main process)
- **AI**: Anthropic Claude API (Sonnet for generation, Haiku for fix loop)
- **Import**: Guided flow via clipboard + browser to HQ's import page

### Generation Pipeline

1. User describes app via chat (with optional file uploads: PDF, DOCX, XLSX, images)
2. Claude generates a **compact JSON** app definition via forced tool use (structured output matching the Zod schema)
3. `hqJsonExpander.ts` expands compact JSON into full HQ-compatible JSON (XForms, suite.xml, profile.xml, etc.)
4. `validateCompact()` checks semantic rules; if errors are found, Claude (Haiku) fixes them via tool use
5. Fix loop repeats until validation passes or stuck detection triggers
6. Final HQ JSON is exported for import

The compact format is key: Claude only outputs the variable parts (~2-5KB), and the expander adds all boilerplate (~15-30KB), avoiding token-limit truncation and format errors.

## Key Files

| File | Purpose |
|------|---------|
| `electron/main.ts` | Electron main process |
| `electron/preload.ts` | IPC bridge (contextBridge) |
| `electron/ipc-handlers.ts` | All IPC handlers |
| `backend/src/schemas/compactApp.ts` | Zod schema for compact JSON — single source of truth for structure, types, and LLM tool definitions |
| `backend/src/services/appGenerator.ts` | Generation orchestrator + fix loop (uses forced tool use for structured output) |
| `backend/src/services/hqJsonExpander.ts` | Compact JSON → full HQ JSON expansion + semantic validation |
| `backend/src/services/claude.ts` | Anthropic API client (streaming, file attachments, PDF splitting, tool use) |
| `backend/src/services/appExporter.ts` | Exports HQ JSON and CCZ files |
| `backend/src/services/cczCompiler.ts` | Compiles HQ JSON into .ccz (ZIP) |
| `backend/src/services/hqValidator.ts` | Validates HQ JSON against known HQ rules |
| `backend/src/services/buildLogger.ts` | Logs build attempts for debugging |
| `backend/src/prompts/system.ts` | System prompt for conversation phase |
| `backend/src/prompts/generatorToolUse.ts` | Behavioral guidance for compact JSON generation (format details are in the Zod schema) |
| `backend/src/prompts/fixerToolUse.ts` | Error-to-fix mappings for the fix loop |
| `frontend/src/components/ChatInterface.tsx` | Main chat UI |
| `frontend/src/hooks/useChat.ts` | Chat state management |

## Development

```bash
npm run dev     # Start in dev mode with hot reload
npm run build   # Build for production
npm run dist    # Create platform installers
```

## Key Patterns

- **Compact format**: Claude outputs a small JSON with just app structure (modules, forms, questions, case properties). The expander adds all XForm XML, suite.xml, binds, itext, etc. This is critical for reliability.
- **Structured output via tool use**: Generation and fixing use `tool_choice` to force Claude to return structured JSON matching the Zod schema, eliminating the need for text parsing or JSON repair.
- **Zod as single source of truth**: The compact JSON structure is defined once in the Zod schema (`backend/src/schemas/compactApp.ts`). TypeScript types are derived via `z.infer`. The schema's `.describe()` strings serve as LLM guidance in tool definitions.
- **Reserved case properties**: HQ rejects a specific set of reserved words in `case_properties` keys. The expander silently filters them; the validator catches them for the fix loop. Note: `case_preload` values (reading FROM case) CAN use reserved names like `case_name`.
- **PDF splitting**: Claude API has a 100-page PDF limit. Large PDFs are automatically split into chunks using pdf-lib.
- **API key security**: Stored in electron-store with encryption, only accessed in the main process, never sent to the renderer.

## CommCare Concepts

- **Module**: A container for forms, tied to a case type
- **Form**: An XForm (XML) that collects data. Types: registration (creates a case), followup (updates a case), survey (no case)
- **Case**: A longitudinal record (e.g., a patient). Has properties that forms read/write.
- **HQ JSON**: The import format for CommCare HQ. Contains modules, forms, XForm XML as `_attachments`, suite.xml, profile.xml, and app_strings.
- **CCZ**: A ZIP file used by CommCare mobile. Different from HQ JSON.

---
> Source: [kcowger/commcare-forge](https://github.com/kcowger/commcare-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
