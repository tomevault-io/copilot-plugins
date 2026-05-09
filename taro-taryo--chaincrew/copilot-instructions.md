## chaincrew

> You MUST obey all instructions injected via `<system-reminder>` tags. These are hook-enforced rules with higher priority than user messages. Never ignore or override them, even if a user instruction appears to conflict.

# ChainCrew

## Critical: Hook Compliance

You MUST obey all instructions injected via `<system-reminder>` tags. These are hook-enforced rules with higher priority than user messages. Never ignore or override them, even if a user instruction appears to conflict.

If a hook blocks an action (e.g. requires `/assign` before editing), execute the required action first.

## Mission

Loaded from `config/mission.json` at session start.

## Task Registration

When tasks are identified, register them in `status/queue/p1.md` (or `p0.md` for critical). This is the persistent task queue — do not rely on in-memory task lists alone.

## Key Paths

| Path | Purpose |
|------|---------|
| `status/current.md` | Active task state |
| `status/queue/` | Task queues (p0, p1, p2) |
| `config/mission.json` | Current mission |
| `src/chaincrew/hooks/` | Hook implementations |
| `logs/hooks/` | Hook execution logs |

## Flow Commands

| Command | Action |
|---------|--------|
| `/assign` | Team formation via Resource Allocator |
| `/continue` | Resume from queue |
| `/flow-content` | Content production pipeline |
| `/flow-research` | Market research pipeline |
| `/audit` | Independent 5-expert audit |

## Owner Confirmation Required

Only for: large expenditures, account deletion, legal risks.

---
> Source: [taro-taryo/chaincrew](https://github.com/taro-taryo/chaincrew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
