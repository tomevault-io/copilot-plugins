## purview-dlm-mcp

> When using the dlm-diagnostics MCP server tools:

# Copilot Instructions — Purview DLM Diagnostics MCP Server

## MCP Tool Selection (Runtime Diagnostics)

When using the dlm-diagnostics MCP server tools:

| User Intent | Tool | Examples |
|-------------|------|----------|
| Reports a problem or error | `run_powershell` | "retention policy not applying", "items not archiving", "policy stuck in Error" |
| Asks how-to or conceptual question | `ask_learn` | "how do I create a retention policy", "what is eDiscovery" |
| Asks to review commands run | `get_execution_log` | "show me what we ran", "summarize the investigation" |

- **Default to `run_powershell`** for anything that sounds like troubleshooting
- **Never use `ask_learn` for active issues** — it returns docs, not diagnostics
- Follow diagnostic guides in `.github/skills/dlm-diagnostics/` when investigating

## Project Overview

MCP server enabling AI assistants to diagnose Microsoft Purview Data Lifecycle Management (DLM) issues in Exchange Online. Provides `run_powershell` (read-only command execution) and `get_execution_log` (audit trail) tools.

## Key Files

| File | Purpose |
|------|---------|
| `src/index.ts` | MCP server entry point, tool registration |
| `src/powershell/executor.ts` | Long-lived pwsh process manager with MSAL auth |
| `src/powershell/allowlist.ts` | Cmdlet allowlist and command validation |
| `src/asklearn.ts` | Purview topic lookup — Learn URLs and guidance |
| `src/tsg-diagnostics.ts` | Pure diagnostic evaluation engine (10 TSGs) |
| `src/logger.ts` | Append-only execution log with Markdown export |

## Security Constraints

- **Read-only only:** Only `Get-*`, `Test-*`, `Export-*` cmdlets are allowed.
- **Blocked verbs:** `Set-*`, `New-*`, `Remove-*`, `Enable-*`, `Start-*`, `Invoke-*`, `Disable-*`, `Stop-*`, `Add-*`, `Clear-*`, `Update-*`, `Register-*`, `Revoke-*`, `Grant-*`.
- **Validation:** Every command is checked against `allowlist.ts` before execution. New cmdlets must be added to `ALLOWED_CMDLETS` explicitly.
- **No stored credentials:** MSAL interactive browser auth only; tokens in-memory.

## Code Style

- **TypeScript ESM** — `"type": "module"`, use `.js` extensions in imports
- **Strict mode** — explicit return types on exported functions
- **Zod** — for MCP tool input schemas
- **Pure evaluators** — diagnostic functions in `tsg-diagnostics.ts` take parsed data, return structured results (no I/O)
- **Copyright header** on every `.ts` file:
  ```
  // Copyright (c) Microsoft Corporation.
  // Licensed under the MIT License.
  ```
- **Naming:** kebab-case files, camelCase variables/functions, PascalCase types/classes

## Build & Test

```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript (tsc)
npm test             # Run tests (vitest run)
npm start            # Start MCP server
```

**Environment variables:** `DLM_UPN`, `DLM_ORGANIZATION` (required), `DLM_COMMAND_TIMEOUT_MS` (optional, default 180000).

## Skills

Diagnostic skills live in `.github/skills/`:
- `dlm-diagnostics/` — 11 troubleshooting guides for DLM issues
- `asklearn/` — Fallback skill for Microsoft Learn documentation lookup
- `skill-creator/` — Meta-skill for authoring new skills

Each skill has a `SKILL.md` and a `references/` directory with per-symptom troubleshooting guides.

## Guidelines for Code Suggestions

1. **Match existing patterns** — follow the structure in `allowlist.ts` for new cmdlets, `tsg-diagnostics.ts` for new evaluators.
2. **Read-only cmdlets only** — never suggest mutating PowerShell commands in diagnostic steps.
3. **Add copyright headers** — include the Microsoft copyright header on new `.ts` files.
4. **Keep evaluators pure** — diagnostic evaluation functions must not perform I/O or side effects.
5. **Update the allowlist** — if a new `Get-*`/`Test-*`/`Export-*` cmdlet is needed, add it to `ALLOWED_CMDLETS` in `src/powershell/allowlist.ts`.
6. **Mirror skills** — new skills must be placed in both `.github/skills/` and `.claude/skills/`.

---
> Source: [microsoft/purview-dlm-mcp](https://github.com/microsoft/purview-dlm-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
