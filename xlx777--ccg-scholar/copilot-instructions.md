## ccg-scholar

> CCG-Scholar 是一个多模型协作科研工作流系统，基于 ccg-workflow 基础设施构建。

# CCG-Scholar

## 项目概述

CCG-Scholar 是一个多模型协作科研工作流系统，基于 ccg-workflow 基础设施构建。

## 技术栈

- **语言**: TypeScript (ES2022, ESNext modules)
- **构建**: unbuild (rollup)
- **CLI**: cac
- **交互**: inquirer
- **配置**: smol-toml
- **国际化**: i18next

## 项目结构

- `src/utils/installer.ts` — 核心安装逻辑，包含 ALL_COMMANDS 和 WORKFLOW_CONFIGS
- `src/types/index.ts` — ModelRouting 类型定义
- `src/cli.ts` — CLI 入口，使用 cac 框架
- `templates/` — 所有模板文件，安装时复制到 `~/.claude/`

## 构建

```bash
npm install
npx unbuild
```

## 安装路径

- 命令: `~/.claude/commands/ccg-scholar/`
- 代理: `~/.claude/agents/ccg-scholar/`
- 技能: `~/.claude/skills/`
- 规则: `~/.claude/rules/`
- 配置: `~/.claude/.ccg-scholar/config.toml`

## 模板变量

模板中使用 `{{VARIABLE}}` 占位符，安装时由 `injectConfigVariables()` 替换：
- `{{LITERATURE_MODELS}}`, `{{ANALYSIS_MODELS}}`, `{{EXPERIMENT_MODELS}}`
- `{{WRITING_MODELS}}`, `{{VISUALIZATION_MODELS}}`, `{{CROSS_REVIEW_MODELS}}`

---
> Source: [xlx777/ccg-scholar](https://github.com/xlx777/ccg-scholar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
