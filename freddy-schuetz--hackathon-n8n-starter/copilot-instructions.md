## hackathon-n8n-starter

> Diese Datei richtet sich an Coding-Agents, die **AGENTS.md** lesen (OpenAI **Codex**, **OpenCode**, Cursor, Gemini CLI, …).

# AGENTS.md — n8n-Hackathon-Starter (für Codex, OpenCode & andere Agents)

Diese Datei richtet sich an Coding-Agents, die **AGENTS.md** lesen (OpenAI **Codex**, **OpenCode**, Cursor, Gemini CLI, …).

> **Du nutzt Claude Code?** Dann brauchst du das hier nicht — nimm **[README.md](README.md)** + **CLAUDE.md** (mit kuratierten Skills + „richte mir das ein"-Setup).

Ziel: n8n-Automatisierungs-Workflows bauen — mit deinem Agent + dem **n8n-mcp**-Server.

## Setup (einmalig)

1. **n8n-Zugang holen:** kostenlose n8n-Cloud-Trial auf [n8n.io](https://n8n.io) starten → Instanz-URL notieren → **Settings → n8n API → API-Key** erstellen.
2. **n8n-mcp anbinden** (läuft per `npx`, keine Installation nötig):

   **Codex** → `~/.codex/config.toml` (global) oder `.codex/config.toml` (projekt-scoped, nur „trusted projects") · Vorlage: [`.codex/config.toml.example`](.codex/config.toml.example)
   ```toml
   [mcp_servers.n8n-mcp]
   command = "npx"
   args = ["-y", "n8n-mcp"]
   [mcp_servers.n8n-mcp.env]
   N8N_API_URL = "https://DEINE-INSTANZ.app.n8n.cloud"
   N8N_API_KEY = "DEIN_KEY"
   MCP_MODE = "stdio"
   ```

   **OpenCode** → `opencode.json` · Vorlage: [`opencode.json.example`](opencode.json.example)
   ```json
   { "mcp": { "n8n-mcp": { "type": "local",
     "command": ["npx", "-y", "n8n-mcp"],
     "environment": { "N8N_API_URL": "https://DEINE-INSTANZ.app.n8n.cloud", "N8N_API_KEY": "DEIN_KEY", "MCP_MODE": "stdio" } } } }
   ```

3. **Verbindung testen:** Agent bitten, `n8n_health_check` aufzurufen → muss OK liefern.
4. ⚠️ **Keys niemals committen** — die echten Configs (`.codex/config.toml`, `opencode.json`) sind in `.gitignore`.

## Wissen & Beispiele

- Das **tiefe n8n-Wissen liefert der n8n-mcp-Server selbst**: starte mit **`tools_documentation()`**, dann `search_nodes` / `get_node` / `search_templates` / `validate_workflow`.
- Importierbare **Lern-Workflows** (mit Sticky-Notes-Erklärungen) in `examples/workflows/`: Grundlagen, KI-Agent, Agent-mit-Tool→Webhook, Agent→Data Table, Mini-Webhook (`hello-webhook`).
- **Frontend/Backend** (optional): lauffähige Beispiele in `frontend-starter/` (Next.js 16, n8n-Webhook + KI-Chat) und `backend-example/` (FastAPI) — Details in deren README. Datenbank-Wahl: `docs/datenbank.md`.
- Hinweis: Die kuratierten Claude-Code-Skills unter `.claude/skills/` werden von Codex/OpenCode **nicht** geladen — die wichtigsten Regeln stehen unten, der Rest kommt über n8n-mcp.

## Standard-Prozess (immer einhalten)

1. `tools_documentation()` 2. **`search_templates()` (Template-First!)** 3. sonst `search_nodes` → `get_node` → `n8n_create_workflow` 4. `n8n_update_partial_workflow` 5. `n8n_validate_workflow` → `n8n_autofix_workflow` 6. **In deiner Instanz TESTEN** (fängt Laufzeitfehler, die der statische Validator nicht sieht!) 7. mit **Sticky Notes** dokumentieren 8. Security prüfen 9. aktivieren 10. bei fertigem/abzugebendem Workflow einen kurzen **Prüfbericht** erstellen (Tests, Validierung, Security-Status).

## Kritische n8n-Konventionen

- **nodeType-Format je Tool:** Search/Validate `nodes-base.*` · Workflow-Tools `n8n-nodes-base.*` · AI/LangChain `@n8n/n8n-nodes-langchain.*`
- **Webhook-Daten unter `.body`:** `{{ $json.body.feld }}` (nicht `{{ $json.feld }}`).
- **Expressions** mit `{{ }}`; in **Code-Nodes ohne** `{{ }}`. Node-Namen mit Leerzeichen: `{{ $node["HTTP Request"].json.x }}`.
- **IF-Node:** zwei Outputs — `branch: "true"` / `branch: "false"` setzen, sonst landen beide am selben Ausgang.
- **addConnection:** vier String-Parameter `source`, `target`, `sourcePort: "main"`, `targetPort: "main"`.
- **AI-Connections:** Sub-Nodes per `ai_languageModel` / `ai_tool` / `ai_memory` an den Agent (nicht `main`).
- **AI-Tool-Node-Namen:** nur Buchstaben/Ziffern/Unterstriche (kein Leerzeichen/Bindestrich/Umlaut, nicht mit Ziffer beginnen) — der Name wird zum LLM-Funktionsnamen.
- **HTTP-Tool für Agents:** `n8n-nodes-base.httpRequestTool` (v4.x) + `$fromAI('feld','Beschreibung','string')` — **nicht** der Legacy `@n8n/n8n-nodes-langchain.toolHttpRequest` (v1.1). Fast jeder Standard-Node kann als Tool dienen.
- **AI-Sub-Nodes haben kein „Execute":** Tool/Modell/Memory laufen nur, wenn der Agent sie aufruft → Workflow über den **Chat** starten, Sub-Node nicht einzeln „Test step".

## Sicherheit

- API-Keys nur via Credentials/Config — **nie** in Node-Parametern oder im Repo.
- Vor Aktivierung: Validierung + Security-Check; keine personenbezogenen Daten in Node-Namen/Notes.

---
> Source: [freddy-schuetz/hackathon-n8n-starter](https://github.com/freddy-schuetz/hackathon-n8n-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
