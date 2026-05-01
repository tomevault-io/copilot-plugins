## claudewiki

> 本文件面向进入仓库工作的 AI agent。

# AGENTS Guide

本文件面向进入仓库工作的 AI agent。

## 1. 当前主线

- 主产品线：`apps/desktop-shell`
- 产品文档入口：`docs/desktop-shell/README.md`
- Rust 集成层入口：`rust/README.md`

## 2. 功能与文档地址

### 仓库概览

- 仓库说明：`README.md`
- 产品总入口：`docs/desktop-shell/README.md`
- Rust / parity 背景：`PARITY.md`

### 当前产品结构

- 架构总览：`docs/desktop-shell/architecture/overview.md`
- 产品文档入口：`docs/desktop-shell/README.md`

### 共享语义与设计 token

- 功能语义：`docs/desktop-shell/tokens/functional-tokens.md`
- 设计 token：`docs/desktop-shell/tokens/design-tokens.md`

### 重构设计与实施过程

- 规格索引：`docs/desktop-shell/specs/README.md`
- 计划索引：`docs/desktop-shell/plans/README.md`
- 桌面壳架构重构设计：`docs/superpowers/specs/2026-04-06-desktop-shell-architecture-refactor-design.md`
- 桌面壳架构重构计划：`docs/superpowers/plans/2026-04-06-desktop-shell-architecture-refactor-plan.md`
- 文档系统设计：`docs/superpowers/specs/2026-04-06-desktop-shell-documentation-record-system-design.md`
- 文档系统计划：`docs/superpowers/plans/2026-04-06-desktop-shell-documentation-record-system-plan.md`

### 开发、验证、维护

- 运维与维护规则：`docs/desktop-shell/operations/README.md`
- Rust workspace：`rust/README.md`

### Rust / parity 边界

- Rust 集成层说明：`rust/README.md`
- parity 背景：`PARITY.md`
- parity 依赖设计：`docs/open-claude-code-parity-dependency-design.md`
- parity 迁移清单：`docs/open-claude-code-parity-migration-checklist.md`

### 历史与迁移资料

- parity 背景：`PARITY.md`
- 依赖设计：`docs/open-claude-code-parity-dependency-design.md`
- 迁移清单：`docs/open-claude-code-parity-migration-checklist.md`

说明：

- `docs/desktop-shell/*` 是当前产品文档主入口
- `specs/`、`plans/` 记录变更过程，不直接等于当前实现真相
- 历史迁移资料不覆盖当前 `architecture/`、`tokens/`、`operations/`

## 3. 修改后的回填位置

- 结构变更：`docs/desktop-shell/architecture/`
- 共享语义 / 设计 token / 交互模型变更：`docs/desktop-shell/tokens/`
- 开发 / 验证 / 维护流程变更：`docs/desktop-shell/operations/`
- 新功能 / 重构方案：`spec` + `plan`
- 稳定落地结果：`architecture` / `tokens` / `operations`

## 4. 文档约束

- 根 `AGENTS.md` 只保留任务路由与规则
- 不在根 `AGENTS.md` 堆积正文知识
- 已迁移文档直接删除旧文件
- 不保留重复正文
- 不制造两个“当前真相”

## 5. 最低验证入口

### Desktop Shell

```bash
cd apps/desktop-shell
npm run build
```

### Desktop Shell Tauri / Rust 集成层

```bash
cd apps/desktop-shell/src-tauri
cargo check
```

### Rust Workspace

```bash
cd rust
cargo check --workspace
```

## 6. Bug 处理原则

- 原因不明确时不猜测
- 优先复现
- 优先补日志或缩小范围
- 先确认真实行为，再决定修复方案

---
> Source: [wangedoo518/claudewiki](https://github.com/wangedoo518/claudewiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
