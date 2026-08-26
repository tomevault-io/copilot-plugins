## yorutsuke-v2

> Local-first AI accounting assistant for second-hand business users. Drag receipts → configurable AI processing (instant/batch/hybrid) → transaction reports.

# Yorutsuke v2 (夜付け)

Local-first AI accounting assistant for second-hand business users. Drag receipts → configurable AI processing (instant/batch/hybrid) → transaction reports.

> "你只管赚钱，记账交给电脑里的 AI"

## AI-First Development Principles

> **This project is designed for AI-assisted development. All code, templates, and patterns prioritize AI readability and reliability.**

### Core Principles

| Principle | Rationale |
|-----------|-----------|
| **Explicit > Abstract** | AI doesn't remember implicit conventions between sessions |
| **Clear > DRY** | AI can generate repetitive code quickly; clarity prevents errors |
| **Concrete > Generic** | Generic types weaken type inference, causing AI mistakes |
| **Simple > Clever** | AI understands straightforward patterns more reliably |

### Guidelines for AI

1. **Copy templates directly** - Don't try to abstract or generalize
2. **Follow explicit patterns** - Each pillar has one recommended way
3. **Check checklists** - Use `.prot/checklists/` before and after coding
4. **Avoid generic factories** - Use explicit functions (e.g., `parseUser` not `createParser`)

See also: `.claude/rules/debugging.md`, `.claude/rules/time-handling.md`

### Work Flow

**MVP → Issues → TODO 的协同机制**

| 文档 | 用途 |
|------|------|
| `.claude/WORKFLOW.md` | 工作流索引 & 速查表 (主入口) |
| `.claude/rules/workflow.md` | 核心原则 (三层架构) |
| `.claude/workflow/planning.md` | Two-Step Planning 详细指南 |

**三层架构**:
- **MVP 文件** (`docs/dev/MVP*.md`)：目标和验收标准（路标）
- **GitHub Issues + Issue Plans** (`.claude/plans/active/`)：详细的技术任务和实现方案（施工图）
- **MEMORY.md** (ADR index only)：长期知识和架构决策（备忘）

**Two-Step Planning**:
1. **Step 1** (40 min): MVP 分解 → 创建 Issues + 依赖图
2. **Step 2** (1-2h): Feature 详细规划 → Dev Plan + Test Cases (just-in-time)

**Templates & Planning**:

| 层级 | Template | Use Case | Command |
|------|----------|----------|---------|
| 战略 | `TEMPLATE-mvp.md` | New MVP release | `*plan` |
| 战役 | `TEMPLATE-feature-plan.md` | Before coding | `*issue pick` |
| 战役 | `TEMPLATE-github-issue.md` | Step 1 issues | `*issue new` |
| 战术 | Issue Plan files | Session work | `.claude/plans/active/#XXX.md` |

Templates location: `.claude/workflow/templates/`
Complete guide: `.claude/workflow/templates/README.md`

---

## Overview

- **Tech Stack**: Tauri 2, React 19, TypeScript, AWS CDK
- **Architecture**: AI_DEV_PROT v15
- **Current Version**: 0.1.0
- **Target Users**: Budget-conscious second-hand computer users (Mercari/Yahoo Auctions sellers)
- **Migration**: 本项目是迁移项目，原项目参考 `../yorutsuke`，参考功能和使用方法，架构完全改变

### Refer to Yorutsuke

参考原项目时的原则：

1. **先看原项目了解"做什么"** - 功能需求、边界条件、已踩过的坑
2. **再按新架构决定"怎么做"** - 遵循 Pillars、分层规范
3. **不直接复制，而是重写** - 适配 AI_DEV_PROT v15 架构

| 参考内容 | 价值 | 说明 |
|----------|------|------|
| Rust 逻辑 | ✅ 高 | 压缩参数、IPC 实现已调优 |
| 业务规则 | ✅ 高 | MD5 去重、quota 限制已验证 |
| 错误处理 | ✅ 高 | 异常场景已踩过坑 |
| 代码结构 | ⚠️ 低 | 架构完全不同，勿复制 |
| React hooks | ⚠️ 低 | 需适配 Pillar L headless |

## Core Flow

```
日間採集 (Day)          処理 (Configurable)         結果確認
──────────────────────────────────────────────────────────────────────────────────
Receipt Drop  →  Local SQLite  →  S3 Upload  →  Nova Lite OCR  →  Transaction Result
     ↓              ↓              ↓                                    ↓
  WebP圧縮      Queue管理   Instant/Batch/Hybrid                  確認/編集
```

**処理方式**: Admin で設定可能（即座処理/バッチ/ハイブリッド）

## Project Structure

| Directory | Purpose |
|-----------|---------|
| **`/app`** | Desktop app (Tauri 2 + React 19) - user-facing receipt capture & sync |
| **`/admin`** | Admin web panel (Vite + React) - operational monitoring & batch config |
| **`/infra`** | AWS CDK - two stacks (App + Admin) + Lambda functions + shared layer |
| **`/docs`** | Source of truth - architecture, design, operations, dev workflow |
| **`/.claude`** | Claude Code config - development rules, workflow templates, active plans |
| **`/.prot`** | AI_DEV_PROT v15 - pillars, checklists, patterns |

## Commands

**NPM Workspaces** - All commands run from root directory:

```bash
# Development
npm run dev:app               # Start Tauri app development (with hot reload)
npm run dev:admin             # Start Admin panel development (http://localhost:5173)

# Building
npm run build                 # Build both app and admin for production

# Deployment
npm run deploy                # Deploy infrastructure + sync environment variables

# Environment Management
npm run env:sync              # Sync CDK outputs to app/.env.local and admin/.env

# Testing
npm run test                  # Run all tests
npm run test:watch            # Watch mode (auto re-run on file change)
npm run test:coverage         # Generate coverage reports

# Workspace-specific commands (if needed)
npm run -w app tauri build    # Build Tauri app directly
npm run -w infra diff         # Preview CDK changes
npm run -w infra synth        # Synthesize CDK template
```

**Lambda development** (TypeScript workflow):
```bash
# Build Lambda TypeScript files (after code changes)
cd infra && npm run build:lambdas          # Compiles .ts → .mjs in .lambda-dist/

# Layer-only deploy (after modifying shared-layer/)
cd infra && npm run build:lambdas && \
zip -r /tmp/layer.zip .lambda-dist/shared-layer/nodejs/ && \
aws lambda publish-layer-version --layer-name yorutsuke-shared-dev --zip-file fileb:///tmp/layer.zip --profile dev
```

## Infrastructure

**Two AWS CDK Stacks**: Main App (S3 → Lambda instant-processor → Bedrock OCR → DynamoDB), Admin Panel (Cognito + API Gateway + CloudFront).

**Lambda Functions**: All 16 Lambda functions migrated to TypeScript (ES2022, strict mode, compiled with esbuild). Shared Lambda layer contains model-analyzer, logger, schemas.

**Build & Deploy**: `npm run deploy` (from root) - automatically builds TypeScript (CDK + Lambdas) before deploying.

## Test Assets

测试图片：`/private/tmp/yorutsuke-test/` → 详见 `docs/tests/TEST_ASSETS.md`

## Workflow

**CRITICAL**: All pull requests must target `development` branch, never `master` directly.

### Core Commands

| Command | Description |
|---------|-------------|
| `*status` | Project overview (git, issues, progress) |
| `*resume` | Resume session (pull, load context) |
| `*sync` | Save progress and sync (commit, push) |
| `*next` | Execute next task |
| `*approve` | Approve pending action |

### Task Management

| Command | Description |
|---------|-------------|
| `*issue` | Manage GitHub issues |
| `*plan` | Create implementation plan |
| `*tier` | Classify task complexity (T1/T2/T3) |
| `*scaffold` | Generate module structure |

### Infrastructure

| Command | Description |
|---------|-------------|
| `*cdk` | AWS CDK operations (status/diff/deploy) |

### Protocol

| Command | Description |
|---------|-------------|
| `*review` | Post-code review |
| `*audit` | Run automated checks |
| `*pillar <X>` | Query Pillar details |

## Complexity Tiers (AI_DEV_PROT v15)

| Tier | Classification | Yorutsuke Example |
|------|---------------|-------------------|
| **T1** | Direct | Fetch transactions, render report |
| **T2** | Logic | Image queue, upload management |
| **T3** | Saga | Batch processing, payment (future) |

## Domain Entities

| Entity | Branded Type | Description |
|--------|--------------|-------------|
| `ImageId` | `string & { __brand: 'ImageId' }` | Receipt image identifier |
| `TransactionId` | `string & { __brand: 'TransactionId' }` | Transaction identifier |
| `UserId` | `string & { __brand: 'UserId' }` | User identifier |

## Architecture Decisions

**Before coding**: Check past decisions → `docs/architecture/ADR/`
- **ADR-001**: Service Pattern (global listeners, Zustand stores)
- **ADR-005**: TraceId vs IntentId (log correlation vs idempotency)
- **ADR-013**: Environment-Based Secret Management (API keys, credentials)

Full list: `docs/architecture/ADR/README.md`

## Rules

**MUST**:
- Classify task tier before implementation
- Use Branded Types for IDs (Pillar A)
- Validate all Lambda responses with Zod (Pillar B)
- Separate UI from logic (Pillar L: Headless) → See ADR-001
- Use Service Pattern for global listeners (not React hooks) → ADR-001
- Add Intent-ID for batch operations (Pillar Q) → See ADR-005
- Load all secrets from environment variables (never hardcode) → See `.claude/rules/secrets.md`
- Publish new Lambda Layer version after modifying shared-layer code → See `.claude/rules/lambda-layer-deployment.md`

**MUST NOT**:
- Use primitive types for domain entities
- Mix JSX in headless hooks
- Trust raw API/IPC responses without parsing
- Skip quota checks before upload
- Commit API keys or credentials to git
- Assume CDK deploy will automatically update Lambda Layer versions

## AWS Resources

| Service | Resource | Purpose |
|---------|----------|---------|
| S3 | `yorutsuke-images-{env}` | Receipt image storage (30-day lifecycle) |
| DynamoDB | `yorutsuke-transactions-{env}` | Transaction records |
| Cognito | `yorutsuke-users-{env}` | User authentication |
| Lambda | `yorutsuke-presign-{env}` | S3 upload URLs |
| Lambda | `yorutsuke-batch-{env}` | Batch Processing (configurable mode) |
| Bedrock | Nova Lite | Receipt OCR (~¥0.015/image) |

## Cost Controls

| Level | Limit | Enforcement |
|-------|-------|-------------|
| Global | ¥1,000/day | Lambda hard stop |
| User | 50 images/day | Quota check |
| Rate | 1 image/2sec | MIN_UPLOAD_INTERVAL_MS |

## Logging

日志位置：`~/.yorutsuke/logs/YYYY-MM-DD.jsonl` → 详见 `docs/operations/LOGGING.md`

```bash
cat ~/.yorutsuke/logs/$(date +%Y-%m-%d).jsonl | jq .  # View today's logs
```

## Design System

**Read before any UI work**: `.claude/rules/design-system.md` + `docs/design/` (Tokens, Components, Views, A11y)

## Memory & Context

- **Active issue plans**: `.claude/plans/active/#XXX.md`
- **Long-term memory**: `.claude/MEMORY.md` (ADR index only, English)
- **Workflow guide**: `.claude/WORKFLOW.md`
- **Design rules**: `.claude/rules/design-system.md` (read before UI coding)
- **Lambda Layer deployment**: `.claude/rules/lambda-layer-deployment.md` + `.claude/rules/lambda-quick-reference.md`
- **Testing guide**: `TEST.md` (quick reference) + `scripts/RUN_TESTS.md` (detailed) + `scripts/QUICK_TEST.md` (examples)

## Key Documentation

| Category | File | Description |
|----------|------|-------------|
| **Architecture** | `docs/architecture/README.md` | Index → LAYERS, PATTERNS, FLOWS, SCHEMA, ADR |
| **Requirements** | `docs/product/REQUIREMENTS.md` | Feature specs, acceptance criteria |
| **MVP Plan** | `docs/dev/MVP_PLAN.md` | Roadmap & current phase |
| **Design System** | `docs/design/` | Tokens, Components, Views, A11y |
| **Operations** | `docs/operations/` | LOGGING.md, QUOTA.md |

**Developer Guides**: `docs/guides/README.md` (see there for Lambda, CDK, Azure DI, and other workflow guides)

---
@.prot/CHEATSHEET.md

<!-- yorutsuke-v2 v0.1.0 -->

---
> Source: [aifuun/yorutsuke-v2](https://github.com/aifuun/yorutsuke-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
