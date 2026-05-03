## pinclaw

> This file provides project-specific guidance for Claude Code. Update this file whenever Claude does something incorrectly so it learns not to repeat mistakes.

# CLAUDE.md - Project Instructions for Claude Code

This file provides project-specific guidance for Claude Code. Update this file whenever Claude does something incorrectly so it learns not to repeat mistakes.

## Project Overview

<!-- Customize this section for your project -->

(Describe your project here)

## Development Workflow

Give Claude verification loops for 2-3x quality improvement:

1. Make changes
2. Run typecheck
3. Run tests
4. Lint before committing
5. Before creating PR: run full lint and test suite

## Code Style & Conventions

<!-- Customize these for your project's conventions -->

- Prefer `type` over `interface`; never use `enum` (use string literal unions instead)
- Use descriptive variable names
- Keep functions small and focused
- Write tests for new functionality
- Handle errors explicitly, don't swallow them

## Commands Reference

```sh
# Verification loop commands (customize for your project)
npm run typecheck        # Type checking
npm run test            # Run tests
npm run lint            # Lint all files
npm run format          # Format code

# Git workflow
git status              # Check current state
git diff                # Review changes before commit
```

## Self-Improvement

After every correction or mistake, update this CLAUDE.md with a rule to prevent repeating it. Claude is good at writing rules for itself.

End corrections with: "Now update CLAUDE.md so you don't make that mistake again."

Keep iterating until the mistake rate measurably drops.

## Working with Plan Mode

- Start every complex task in plan mode (shift+tab to cycle)
- Pour energy into the plan so Claude can 1-shot the implementation
- When something goes sideways, switch back to plan mode and re-plan. Don't keep pushing.
- Use plan mode for verification steps too, not just for the build

## Parallel Work

- For tasks that need more compute, use subagents to work in parallel
- Offload individual tasks to subagents to keep the main context window clean and focused
- When working in parallel, only one agent should edit a given file at a time
- For fully parallel workstreams, use git worktrees:
  `git worktree add .claude/worktrees/<name> origin/main`

## Things Claude Should NOT Do

<!-- Add mistakes Claude makes so it learns -->

- Don't use `any` type in TypeScript without explicit approval
- Don't skip error handling
- Don't commit without running tests first
- Don't make breaking API changes without discussion
- **绝对不要在没有查证的情况下编造 Pinclaw 项目的技术细节**。对比其他项目时，必须先读本仓库代码确认事实，不能凭印象或推测填写。Pinclaw 用的是 nRF52840（XIAO BLE Sense），不是 ESP32。
- **不要想当然**。涉及硬件型号、芯片、协议等具体技术事实，必须 grep/read 代码验证后再写，宁可说"我不确定"也不能编。

## Mode Isolation: Pinclaw AI vs MyOpenClaw (CRITICAL)

Pinclaw has TWO completely independent operational modes. **They MUST NEVER share code paths, data, or routing logic.**

### Pinclaw AI (fastclaw) — Cloud mode

- **WS Handler:** `cloud/api/src/routes/ws-handlers/fastclaw-handler.ts`
- **Services:** `fastclaw-proxy.ts`, `fastclaw-cron-manager.ts`, `fastclaw-agent-manager.ts`
- **Dashboard (website) ALWAYS uses this mode**
- **DB instance_type:** `'fastclaw'`

### MyOpenClaw (relay) — Local OpenClaw mode

- **WS Handler:** `cloud/api/src/routes/ws-handlers/relay-handler.ts`
- **Services:** `relay-service.ts`, `offline-queue.ts`
- **Dashboard NEVER uses this mode**
- **DB instance_type:** `'relay'`

### MyHermes (hermes) — Local Hermes / OpenAI-compatible mode

- **WS Handler:** `cloud/api/src/routes/ws-handlers/hermes-handler.ts`
- **Bridge:** `bridge/` (standalone npm package `pinclaw-bridge`)
- **Uses relay-service.ts tunnel infrastructure** (shared with MyOpenClaw)
- **Dashboard NEVER uses this mode**
- **DB instance_type:** `'hermes'`

### HARD RULES — violating these causes production bugs:

1. **fastclaw-handler.ts MUST NEVER import from `relay-service.ts` or `offline-queue.ts`**
2. **relay-handler.ts MUST NEVER import from `fastclaw-proxy.ts` or `fastclaw-*.ts`**
   2b. **hermes-handler.ts MUST NEVER import from `fastclaw-*.ts` or `relay-handler.ts`**
3. **gateway-rpc.ts:** `chat.send` ALWAYS routes to FastClaw. Other methods check instance_type.
   3b. **Dashboard API queries (instances/me, chat-api, etc.) MUST filter `instance_type = 'fastclaw'`** — NEVER use `ORDER BY created_at DESC LIMIT 1` without filtering instance_type, because a newer relay instance will shadow the fastclaw instance and break the entire Dashboard.
4. **All data stores must include mode:** `storeMessage()` requires `instanceType`, `storeContextHints()` requires `connectionMode`, `device_skills_store` is per-instanceId
5. **When adding new features:** decide which mode it belongs to → put it in the correct handler file
6. **`unified-ws.ts` is a thin router ONLY.** Mode-specific logic goes in the handler files.
7. **本地跑的 OpenClaw 永远不会出现在 Pinclaw AI 的 Dashboard 里。**
8. **绝不能把 fastclaw 实例的 instance_type 改成 relay，反之亦然。** 修改数据库 instances 表时必须保持 instance_type 不变。subdomain 前缀 `f-` 永远对应 fastclaw，`u-` 永远对应 relay。混淆会导致连接认证失败（subdomain 唯一约束冲突 → 用户无法连接）。
9. **清理消息时注意 iOS 端会自动重发。** iOS 的 `ChatViewModel` 在 WebSocket 重连后会自动重发所有本地"未确认"的用户消息（无 TTL 上限），删除服务器消息会触发大规模重发。清理前必须先停 FastClaw，清理后确认 iOS 端杀掉 App 再重启。

### Connection mode strings:

| Layer                           | Pinclaw AI   | MyOpenClaw     | MyHermes     |
| ------------------------------- | ------------ | -------------- | ------------ |
| Client sends                    | `"cloud"`    | `"myopenclaw"` | `"myhermes"` |
| Server internal                 | `"fastclaw"` | `"relay"`      | `"hermes"`   |
| Server responds                 | `"cloud"`    | `"myopenclaw"` | `"myhermes"` |
| DB instance_type                | `"fastclaw"` | `"relay"`      | `"hermes"`   |
| ClientConnection.connectionMode | `"fastclaw"` | `"relay"`      | `"hermes"`   |
| Subdomain prefix                | `f-`         | `r-` / `u-`    | `h-`         |

## Deployment Safety (CRITICAL)

**Every exported function that is `import`-ed must exist.** The API uses `tsx` (no compile step), so missing exports crash the ENTIRE server at startup with no warning during build. This has caused production outages.

### HARD RULES for deployment:

1. **Never add an import without implementing the export.** If you add `import { foo } from "./bar.js"`, `foo` MUST be exported from `bar.ts` in the SAME commit.
2. **Before deploying**: run `node --import tsx -e "import('./src/server.ts')"` in the api directory to verify all imports resolve. A broken import = full server crash loop.
3. **After deploying**: always check `docker logs pinclaw-cloud-control-plane-1 --tail 20` to verify the process started without import errors.
4. **FastClaw instance provisioning**: use `provisionFastClawInstance()` (DB record only, no Docker container). NEVER use `provisionInstance()` or `provisionManagedInstance()` for fastclaw — those create OpenClaw Docker containers with wrong `instance_type`.

## Project-Specific Patterns

- The WebSocket endpoint `/ws` is a **routing layer** (`unified-ws.ts`), not merged logic
- `ClientConnection.connectionMode` (not `instanceType`) stores the mode per connection
- FastClaw is a first-class DB instance type (migration 031), not a virtual/inferred type

---

_Update this file continuously. Every mistake Claude makes is a learning opportunity._

---
> Source: [ericshang98/pinclaw](https://github.com/ericshang98/pinclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
