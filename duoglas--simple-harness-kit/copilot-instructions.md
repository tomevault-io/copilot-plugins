## simple-harness-kit

> 兼容 OpenAI Codex、Cursor 及其他支持 AGENTS.md 标准的工具。

# AGENTS.md

兼容 OpenAI Codex、Cursor 及其他支持 AGENTS.md 标准的工具。

## 项目概述

Simple Harness Kit — 可移植的 Harness Engineering 方法论 + 模板仓库。阅读 `methodology/` 了解完整框架，阅读 `init-prompt.md` 了解如何为新项目初始化 Harness。

## 约定

- 文档默认中文；代码示例和技术术语保留英文
- 方法论文档在 `methodology/`，编号 00-10
- 模板在 `templates/`，使用 `.tmpl` 后缀
- Skills 遵循 SKILL.md 格式（YAML frontmatter）

## 核心工作流

本仓库定义了 6 阶段开发循环：Plan → Setup → Execute → Verify → Review → Feedback。每个阶段有 Gate 条件，满足后才能进入下一阶段。

## 使用本仓库

- 理解方法论：从 `methodology/00-philosophy.md` 开始，按编号顺序阅读
- 为项目生成 Harness：阅读 `init-prompt.md` 并按指引操作
- 安装 Skills：将 `skills/*/` 复制到你的 skills 目录

---
> Source: [duoglas/simple-harness-kit](https://github.com/duoglas/simple-harness-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
