## outbound-guardian

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Outbound Guardian is a Thunderbird 148 MailExtension (Manifest V3) that intercepts email sending to run user-defined LLM-powered checks. If any check triggers, the send is cancelled and a warning dialog appears.

Each check rule consists of:
1. **Condition prompt** (required): Boolean check — LLM returns `{ triggered: true/false, reason: string }`
2. **Improvement prompt** (optional): Auto-fix instruction — LLM returns `{ improvedBody: string, changes: string[] }`

## Architecture

### Data Flow
```
Send clicked → onBeforeSend → get compose details → run rules through LLM →
  all pass → send proceeds
  any fail → cancel send → open warning dialog →
    Edit / Auto-fix / Send Anyway
```

### Key Modules
Background scripts are loaded as classic scripts (not ES modules) via `manifest.json` `background.scripts` array. They share a global scope — no `import`/`export`. Load order matters: `prompts.js` → `provider.js` → `engine.js` → `background.js`.

- **`src/background.js`** — Entry point. Registers `messenger.compose.onBeforeSend` listener. Orchestrates rule checking and dialog lifecycle. Uses a `bypassTabs` Set to prevent infinite loops when "Send Anyway" re-triggers send.
- **`src/llm/provider.js`** — LLM API abstraction. MVP: OpenAI Responses API (`POST /v1/responses`) with structured output (`text.format: json_schema`). Global functions: `checkRule(email, rule, settings)` and `improveEmail(email, rule, settings)`.
- **`src/llm/prompts.js`** — Constructs instructions + user content for the Responses API. Global functions: `buildCheckPrompt()`, `buildImprovePrompt()`. Returns `{ instructions, userContent }`.
- **`src/rules/engine.js`** — Iterates enabled rules, calls LLM provider, returns triggered results. Global function: `evaluateRules()`.
- **`src/dialog/`** — Warning popup opened via `messenger.windows.create()`. Communicates with background via `messenger.runtime.sendMessage/onMessage`.
- **`src/options/`** — Settings page for API config and rule CRUD. Accessed via Add-ons Manager.

### Communication Patterns
- **Background ↔ Dialog**: `messenger.runtime.sendMessage` / `messenger.runtime.onMessage`
- **Background ↔ Options**: Both read/write `messenger.storage.local`; background watches `storage.onChanged`
- **Data passing to dialog**: Triggered results stored in `messenger.storage.local` under a temp key; dialog reads on load

### Storage Keys
- `settings` — `{ provider, apiKey, model, enabled }`
- `rules` — Array of `{ id, name, conditionPrompt, improvementPrompt, enabled, order }`

## Thunderbird Extension Conventions

- Use `messenger.*` namespace (not `browser.*`) for all WebExtension APIs
- Manifest V3 with `"strict_min_version": "128.0"`
- All APIs are async/Promise-based
- Background scripts are persistent (load at startup)
- Host permissions required for each LLM API domain in `manifest.json`

## Build & Test

### Load for Development
1. Open Thunderbird
2. Go to **Tools → Add-ons and Themes** (or `about:addons`)
3. Click gear icon → **Install Add-on from File...**
4. Select `manifest.json` from the project root

### Debug
- **Error Console**: Tools → Developer Tools → Error Console
- Background script logs appear in the Error Console
- Dialog/options page: right-click → Inspect in their respective windows

### Package for Distribution
```bash
cd /c/dev/prj/outbound-guardian
zip -r outbound-guardian.xpi manifest.json src/ icons/ -x "*.git*"
```

## LLM Provider Strategy

- **MVP**: OpenAI Responses API (`POST /v1/responses`, `text.format: { type: "json_schema" }`)
- **Later**: OpenAI-compatible generic endpoint (custom base URL, responses API + chat completions), Anthropic Messages API, Google Gemini API
- Provider interface: `checkRule(emailContent, rule) → { triggered, reason }` and `improveEmail(emailContent, rule) → { improvedBody, changes }`
- All providers must return the same response shape regardless of underlying API differences

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stimpy77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
