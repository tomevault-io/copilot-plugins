## lingyun

> LingYun is a VS Code extension that provides an agentic AI assistant + a tool framework.

# AGENTS.md (LingYun)

LingYun is a VS Code extension that provides an agentic AI assistant + a tool framework.
This file is loaded into the agent’s system prompt (see `packages/vscode-extension/src/core/instructions.ts`), so keep it **accurate, compact, and actionable**.

## Development Policy (Important)

This project is still in active development with no production users. Prefer breaking changes when they reduce complexity and lead to a cleaner architecture.

- No legacy compatibility: delete/replace old paths instead of layering.
- Minimal duplication: centralize shared logic (prompt shaping, tool formatting, approvals, streaming glue).
- Persisted state: no migrations; users/devs can clear via `LingYun: Clear Saved Sessions`.
- No sensitive info: never log/write API keys, private base URLs, or internal IPs.

## Current Architecture (Source of Truth)

- Extension entry + commands + public API: `packages/vscode-extension/src/extension.ts`
- Agent core (OpenCode-aligned “assistant message + parts” history; ai-sdk streaming + tool calls): `packages/vscode-extension/src/core/agent/index.ts`
- Tool registry / providers: `packages/vscode-extension/src/core/registry.ts`
- LLM providers:
  - GitHub Copilot: `packages/vscode-extension/src/providers/copilot.ts`
  - OpenAI-compatible server: `packages/vscode-extension/src/providers/openaiCompatible.ts`
- Workspace tools:
  - `.vscode/agent-tools.json` and `.vscode/agent-tools/*.json` (loaded by `packages/vscode-extension/src/providers/workspace.ts`)
  - Schema: `packages/vscode-extension/schemas/agent-tools.schema.json`
- Built-in tools (stable IDs): `packages/vscode-extension/src/tools/builtin/*`
  - `read`, `read_range`, `write`, `edit`, `glob`, `grep`, `list`, `lsp`, `symbols_search`, `symbols_peek`, `get_memory`, `update_memory`, `bash`
- Memory pipeline + artifacts: `packages/vscode-extension/src/core/memories/index.ts`
- Chat UI:
  - Webview provider: `packages/vscode-extension/src/ui/chat.ts` + `packages/vscode-extension/src/ui/chat/methods.*.ts`
  - Webview assets: `packages/vscode-extension/media/chat.html`, `packages/vscode-extension/media/chat/*.js`
- Headless Agent SDK (Node runtime): `packages/agent-sdk/`
- Shared core library (WIP): `packages/core/`

## Modes (OpenCode-style)

- Plan mode (`lingyun.mode=plan`): read-only. Edit/write are denied; auto-approve is disabled.
- Build mode (`lingyun.mode=build`): full tool use with approvals based on tool metadata + settings.
- “Plan first” flow is controlled by `lingyun.planFirst` in the UI layer.

## Safety Model (Approvals + Workspace Boundaries)

- Tools declare safety via `ToolDefinition.metadata` (`permission`, `permissionPatterns`, `requiresApproval`, `readOnly`).
- `lingyun.autoApprove` (Build mode only) skips approval prompts.
- External paths are blocked by default, even with `autoApprove`:
  - Setting: `lingyun.security.allowExternalPaths` (default `false`).
  - Enforced both in the agent permission layer and in path resolution (`resolveToolPath`).

## Sessions + Persistence

- Session persistence is enabled by default: `lingyun.sessions.persist` (default `true`).
- Stored under VS Code extension storage (`context.storageUri/sessions/`, fallback `context.globalStorageUri/sessions/`).
- Schema is hard-reset on mismatch (no migrations).

## Context Window / Compaction

- Token tracking + prune/compaction live in `packages/vscode-extension/src/core/compaction.ts`.
- Settings: `lingyun.compaction.*` + optional per-model limits `lingyun.modelLimits`.
- UI should show compaction as a global operation (not “thinking” attached to an old turn).

## Debugging

- `lingyun.debug.details`: enables fuller output-channel logs and acts as a global debug switch for LLM/tool/plugin logging; paths/URLs/hosts stay visible, but credential-like secrets remain masked.
- `lingyun.debug.llm`: logs redacted request/usage metadata to the “LingYun” output channel.
- `lingyun.debug.tools`: logs tool execution/approval boundary events (no prompts/URLs).
- `lingyun.debug.plugins`: logs plugin discovery/loading events.
- `LingYun: Show Logs` opens the output channel.

## Minimal Local Model Config (Example)

```json
{
  "lingyun.llmProvider": "openaiCompatible",
  "lingyun.openaiCompatible.baseURL": "http://localhost:8080/v1",
  "lingyun.openaiCompatible.defaultModelId": "your-model-id",
  "lingyun.maxOutputTokens": 32000
}
```

---
> Source: [mzbac/lingyun](https://github.com/mzbac/lingyun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
