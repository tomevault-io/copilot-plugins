## clawless

> Bridge connecting local AI agent CLIs (Gemini CLI, OpenCode, Claude Code) to Telegram/Slack via ACP.

# Agent Context for Clawless

Bridge connecting local AI agent CLIs (Gemini CLI, OpenCode, Claude Code) to Telegram/Slack via ACP.

**Core**: BYO-agent — swap CLI runtimes without rebuilding bot integration.

## Supported Agents

| Agent | CLI Command | Config Value |
|-------|-------------|--------------|
| Gemini CLI | `gemini` | `gemini` (default) |
| OpenCode | `opencode` | `opencode` |
| Claude Code | `claude` | `claude` |

## Architecture

```
Messaging (Telegram/Slack) → Clawless Bridge → Agent CLI (ACP)
                                      ↓
                              Scheduler + Callback API
```

**Components**: `messaging/` (clients), `acp/` (session mgmt), `core/agents/` (CLI interface), `scheduler/` (cron jobs), `utils/` (memory/history)

## Async Mode

- **QUICK**: Simple questions, answered immediately
- **ASYNC**: Tasks needing tools or >10s — returns confirmation, executes in background

## Development

```bash
npm run dev      # Watch mode
npm run build    # lint + format + compile
npm run test     # vitest
```

**Stack**: TypeScript (ESM), Node.js 18+, Biome, Vitest, Pino logging, Zod validation

## Config & Persistence

All in `~/.clawless/`: `config.json`, `MEMORY.md`, `conversation-history.jsonl`, `conversation-semantic-memory.db`, `schedules.json`

## API Endpoints (port 8788)

| Endpoint | Purpose |
|----------|---------|
| `GET /health` | Health check |
| `POST /callback/:platform` | Proactive notifications |
| `POST /api/message` | Send to bound chat |
| `GET/POST/PATCH/DELETE /api/schedule` | Cron job management |
| `POST /api/memory/semantic-recall` | Semantic memory retrieval |

## Key Files

| Area | Files |
|------|-------|
| Entry | `index.ts`, `bin/cli.ts` |
| App | `app/ClawlessApp.ts`, `app/AgentManager.ts` |
| Agents | `core/agents/*.ts` |
| ACP | `acp/runtimeManager.ts`, `acp/tempAcpRunner.ts` |
| Messaging | `messaging/telegramClient.ts`, `messaging/slackClient.ts`, `messaging/ModeDetector.ts` |
| Scheduler | `scheduler/*.ts` |
| Utils | `utils/config.ts`, `utils/memory.ts`, `utils/semanticConversationMemory.ts` |

## Debug Flags

- `ACP_DEBUG_STREAM=true` — ACP stream debug logs
- `LOG_LEVEL=debug` — Debug logging

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HainanZhao)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HainanZhao)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
