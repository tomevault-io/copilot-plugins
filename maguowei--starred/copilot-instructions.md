## starred

> 本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 项目概述

`starred` 是一个 Python CLI 工具,从用户的 GitHub stars 生成并维护 Awesome 风格的列表。源码位于 `starred/`:`starred.py`(Click CLI 入口)和 `githubgql.py`(GitHub GraphQL 客户端)。

## 环境

- 包管理器使用 **`uv`**(已从 poetry 迁移)。安装依赖:`uv sync --dev`。
- 运行或测试 CLI 需要 `GITHUB_TOKEN`(GitHub personal access token)。

## 命令

- Lint:`uv run pycodestyle --max-line-length=200 starred`
- 构建:`uv build`
- 运行 / 集成测试:`uv run starred --username <user> --token ${GITHUB_TOKEN} --sort`

## 代码风格

- 使用 pycodestyle,行长限制为 **`--max-line-length=200`**(非 pycodestyle 默认的 79)。CI 严格执行此命令,代码行需控制在 200 字符内。
- 未配置自动格式化工具(无 black/ruff),请沿用现有风格。

## 测试

- 无单元测试。验证标准为上述集成测试:使用有效的 `GITHUB_TOKEN` 针对真实 GitHub API 运行(CI 即运行此命令)。

## 工作流

- 在 feature 分支上开发,向 `main` 提 PR。
- 提交信息遵循 Conventional Commits 规范。

---
> Source: [maguowei/starred](https://github.com/maguowei/starred) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
