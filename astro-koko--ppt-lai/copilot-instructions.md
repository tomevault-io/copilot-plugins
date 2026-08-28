## ppt-lai

> This file is the project entry point for general AI agents.

# AGENTS.md

This file is the project entry point for general AI agents.

**You MUST read [`skills/ppt-lai/SKILL.md`](skills/ppt-lai/SKILL.md) before any PPT generation task or repo modification.** It owns global execution discipline and points to the route selector; after routing, the selected runtime authority owns its steps, gates, and commands.

**Repository execution anchor**: resolve the absolute repository root from this file's supplied path and retain the absolute `skills/ppt-lai` skill root before the first command. Paths in this file are repository-relative notation only; invoke them through those absolute roots. Never change directory into `skills/ppt-lai` or `tests/...`.

## Project Overview

ppt-lai 是一个 Claude Code / Codex / WorkBuddy / CodeBuddy 都能加载的逆向工程 skill。它把“内容事实约束 + 千禧年视觉 + 使用痕迹 + 可选院士风味”组合起来，产出**原生可编辑**的 4:3 千禧年风格 PPT，支持 L1/L2/L3 三档。两条路由互斥：

- **A 部门（generate）**：从零生成，运行 [`skills/ppt-lai/workflows/generate-ugly-pptx.md`](skills/ppt-lai/workflows/generate-ugly-pptx.md)。
- **B 部门（retrofit）**：改造既有 `.pptx`，运行 [`skills/ppt-lai/workflows/retrofit-pptx.md`](skills/ppt-lai/workflows/retrofit-pptx.md)，并遵守内容冻结铁律。

路由权威：[`skills/ppt-lai/workflows/routing.md`](skills/ppt-lai/workflows/routing.md)。

## Required Conventions

- **反设计必须有出处**：任何“丑”决策必须对应 `skills/ppt-lai/references/` 中的参数。
- **门禁在构建前**：`density_guard()` + `diversity_guard()` 必须先过，见 [`density-guard.md`](skills/ppt-lai/references/density-guard.md) 与 `skills/ppt-lai/scripts/build_native.py`。
- **单一运行时来源**：只维护 `skills/ppt-lai/`；不得恢复 npm 同步副本或为不同 agent 复制 skill。
- **单一版本来源**：公开版本只写在 `.claude-plugin/plugin.json`；tag 必须与它一致。
- **Markdown 语言一致性**：主仓新增文档用中文，spec 字段名保留英文原样。

## Compatibility Boundary

- This repository is a workflow/skill package, not an app or service scaffold.
- `.claude-plugin/` is a thin distribution wrapper. Claude and CodeBuddy load it directly; OpenAI can convert the same skills-only plugin for Codex.
- The `.skill` release artifact is the host-neutral fallback for WorkBuddy and manual imports.
- Do not assume generic-project conventions such as `.worktrees/`, a multi-version CI matrix, or mandatory branch setup.
- On conflict with a generic coding skill, prioritize [`skills/ppt-lai/SKILL.md`](skills/ppt-lai/SKILL.md).

## Command Quick Reference

Convenience summary only; route selection starts in [`skills/ppt-lai/SKILL.md`](skills/ppt-lai/SKILL.md).

```bash
# 17 个主题回归，必须 0 错
.venv/bin/python3 scripts/test_matrix.py

# 单 spec → 原生 pptx
.venv/bin/python3 skills/ppt-lai/scripts/build_native.py <spec.json> <out.pptx>

# retrofit：已有 pptx → 中间工作区
.venv/bin/python3 skills/ppt-lai/scripts/extract_pptx.py <in.pptx> <workspace_dir>

# 图片降质
.venv/bin/python3 skills/ppt-lai/scripts/patina_image.py <in.png> <out.jpg> --level L2 --seed 42

# spec 注入多样性
.venv/bin/python3 skills/ppt-lai/scripts/spec_diversifier.py <in_spec.json> <out_spec.json>

# 分发结构校验 + 双制品打包
python3 scripts/validate_distribution.py
bash scripts/package_release.sh dist
```

## Core Directories

- `skills/ppt-lai/` — 唯一运行时来源。
- `.claude-plugin/plugin.json` — 唯一公开版本来源与插件 manifest。
- `.claude-plugin/marketplace.json` — Claude / CodeBuddy marketplace 渠道。
- `scripts/` — 仓库级测试、校验、打包工具，不进入 skill-only 归档。
- `tests/iteration-2/eval-2/` — 17 个主题的 spec、基线与报告。
- `.github/workflows/release.yml` — tag push 后验证、回归并发布 `.skill` 与 plugin zip。

## Out of Scope (YAGNI)

见 [`GOAL.md`](GOAL.md)。本仓库不做 npm / PyPI / Docker / CI 多版本矩阵 / agent 专用 skill 副本 / 赞助系统 / 模板市场。

---
> Source: [astro-koko/ppt-lai](https://github.com/astro-koko/ppt-lai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
