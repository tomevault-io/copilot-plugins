## vibe-coding-cn

> - 根目录：`README.md` 给出全貌，`Makefile` 封装日常命令，`CONTRIBUTING.md` 说明贡献流程，`LICENSE` 载明协议。保持根目录扁平，避免巨石文件。

# Repository Guidelines

## Project Structure & Module Organization
- 根目录：`README.md` 给出全貌，`Makefile` 封装日常命令，`CONTRIBUTING.md` 说明贡献流程，`LICENSE` 载明协议。保持根目录扁平，避免巨石文件。
- 多语言 i18n：`i18n/<lang>/` 统一三层结构（documents / prompts / skills）。现有语言：中文 `zh`、英文 `en`、希伯来语 `he`，以及高频/常用的 `es`、`hi`、`ar`、`pt`、`ru`、`fr`、`de`、`ja`、`ko`、`it`、`tr`、`nl`、`pl`、`id`、`vi`、`th`、`fa`、`uk`、`bn`、`ta`、`ur`、`ms`、`sw`、`ha`；新增语言遵循同样层级。
- 文档库：`i18n/zh/documents/` 仍是默认中文方法论入口（如 `i18n/zh/documents/Templates and Resources/代码组织.md` 等），新增语言文档按需在对应 `documents/` 下创建并保持同步。
- 提示词资产：`i18n/zh/prompts/` 按角色拆分（system / assistant / coding / user），`libs/external/prompts-library/` 提供 Excel ↔ Markdown 互转工具与脚本目录，便于批量维护提示词，适合作为“单一真实来源”。
- 代码与集成：`libs/` 预留核心实现骨架，`common/`、`database/`、`external/` 分别对应通用模型、存储适配与外部依赖登记；新增模块需保持分层边界与单一职责，避免跨层调用。
- 备份：`backups/` 内含 `一键备份.sh` 与 `快速备份.py`，用于本地快照或同步，请先在隔离目录试跑，确认输出路径与权限。

## Build, Test, and Development Commands
- `make help`：列出所有 Make 目标，是新人快速上手的入口。
- `make lint`：使用 `markdownlint-cli` 校验全仓库 Markdown，一旦新增文档请先跑通（需本地 Node/npm 环境，可用 `npm install -g markdownlint-cli` 安装）。
- `make build` / `make test` / `make clean`：目前为占位，落地具体实现后务必更新脚本和说明；建议在 `Makefile` 旁补充注释并保持幂等，避免修改全局状态。
- 提示词转换：进入 `libs/external/prompts-library/` 后执行 `python main.py` 按交互提示进行转换，运行前请确认虚拟环境、依赖与输出目录，并在完成后检查生成 Markdown 是否符合 lint 规则。

## Coding Style & Naming Conventions
- 文字层：文档、注释、日志使用中文；代码符号（函数 / 变量 / 模块）统一英文且语义直白，避免晦涩缩写。
- 缩进与排版：全仓保持空格缩进（2 或 4 空格任选其一但不得混用）；Markdown 列表、代码块与表格对齐清晰，行宽控制在 120 列内。Git diff 可读性优先。
- 设计品味：优先消除分支与重复；函数力求单一职责且短小；命名遵循小写加中划线或下划线，不使用空格与特殊字符；跨模块接口保持稳定签名。
- 依赖管理：新增工具或库时记录安装方式、最小版本与来源，必要时在 `i18n/zh/documents/Templates and Resources/工具集.md` 或 README 中补充，并说明为何需要它（性能、兼容、功能）。

## Testing Guidelines
- 当前无实测用例；引入代码时请至少提供最小可复现测试。推荐 Python 使用 `pytest`，文件命名 `test_*.py`，夹具精简可读，遵循 red-green-refactor 循环。
- 文档与提示词改动：提交前运行 `make lint`；如转换脚本涉及数据，附带示例输入 / 输出说明或最小数据样例，确保可重复。
- 覆盖率基线由模块维护者设定；若暂无标准，确保主流程和边界条件均可被测试验证，必要时在 PR 描述中写明未覆盖的风险，并建议后续补测计划。

## Commit & Pull Request Guidelines
- Commit 建议遵循简化 Conventional Commits：`feat|fix|docs|chore|refactor|test: scope – summary`，一句话说明行为与范围；避免笼统的 “update”。
- PR 必填：变更摘要、动机或关联 Issue、测试与验证步骤（列出运行的命令与结果概览）；涉及文档 / UI 的修改应附对比截图或链接，方便 reviewer 快速复核。
- 提交前清单：跑通 `make lint`；若新增脚本 / 依赖，更新对应文档与 `Makefile` 目标；确认不携带临时文件或机密数据，并在描述中注明潜在风险或需要 reviewer 特别关注的点。

## Security & Configuration Tips
- 运行备份或转换脚本前，确认输出目录不会覆盖私有数据；建议先在临时目录试跑并检查生成文件，必要时使用只读副本。
- 外部依赖来源记录在 `libs/external/` 目录下，增减依赖时同步维护，保持可追溯；引入第三方脚本需标明许可证与来源。

## Architecture Overview & Workflow
- 工作流倡导「规划 → 上下文固定 → 分步实现 → 自测 → 复盘」，对应资产分别存放在 `i18n/zh/documents/`、`i18n/zh/prompts/`、`libs/` 与备份脚本中。保持单向数据流和清晰责任边界可以避免后期维护成本激增。
- 设计决策与目录结构更新后，请同步修订本文件与相关文档，确保团队共享同一真相源，减少口头约定与隐式规则。

---

# CLAUDE.md

This file provides guidance to Claude series models when working with code in this repository.

## Repository Overview

This is the **Vibe Coding CN** repository, a workflow, toolset, and knowledge base for advanced AI-assisted programming. The project's core assets are its extensive `prompts` and `skills` libraries.

## Key Commands

### Prompt Library Management
```bash
# Enter the library directory
cd libs/external/prompts-library

# Run the interactive conversion tool
python3 main.py
```

### Development & Maintenance
```bash
# Lint all markdown files in the repository
make lint

# Create a full project backup (respects .gitignore)
bash backups/一键备份.sh
```

## Architecture & Structure

### Core Directories
- **`i18n/zh/prompts/`**: The core asset. A massive, well-organized library of prompts.
  - `coding_prompts/`, `system_prompts/`, `user_prompts/`, `meta_prompts/`
- **`i18n/zh/skills/`**: A modular library of skills for the AI, providing domain-specific knowledge for various tools like `ccxt`, `postgresql`, `telegram-dev`, 
etc.
- **`i18n/zh/documents/`**: The project's knowledge base, containing methodology, principles, and guides.
- **`libs/external/prompts-library/`**: A Python-based tool for converting prompts between Excel and Markdown formats.
- **`backups/`**: Scripts for project backups.
- **`libs/`**: Skeleton for shared Python library code.

### Key Technical Details
1.  **Prompt Organization**: Prompts use a `(row,col)_` prefix for categorization.
2.  **Conversion Tool**: The `prompts-library` uses Python with `pandas` and `openpyxl`.
3.  **Documentation Standard**: User-facing documentation is in Chinese. Code, file names, and structure are in English.
4.  **Skills**: The `skills` directory provides context and knowledge for specific tools and domains, each with its own `SKILL.md`.

## Development Workflow

When modifying this repository:
1.  Follow the existing prompt and skill categorization systems.
2.  Use the `prompts-library` tool to maintain consistency when updating prompts.
3.  Run `make lint` after changing any Markdown files.
4.  Run a backup with `bash backups/一键备份.sh` before any major refactoring.

---

# GEMINI.md - 项目上下文文档 (Project Context Document)

## 项目概述 (Project Overview)

`vibe-coding-cn` 项目旨在提供一个通过与 AI 结对编程实现“将想法变为现实”的终极工作流程。它强调“规划驱动”和“模块化”的核心理念，旨在避免 AI 失控导致的项目混乱。该项目不仅仅是一个代码库，更是一个全面的 AI 结对编程指南、庞大的提示词库和模块化的技能工具集，涵盖了从项目构思、技术选型、实施规划到具体开发、调试和扩展的全过程。

**核心理念:** 规划是核心，通过结构化、模块化的方式引导 AI，确保项目可控、可维护。

## 技术栈 (Technology Stack)

本项目主要的技术栈和相关工具包括：

*   **核心语言:** Python (用于 `prompts-library` 工具和备份脚本)
*   **CLI 交互:** `rich`, `InquirerPy` (用于 `prompts-library` 提供友好的命令行界面)
*   **数据处理:** `pandas`, `openpyxl` (用于 `prompts-library` 处理 Excel 文件)
*   **配置管理:** `PyYAML` (用于 `prompts-library` 的配置)
*   **文档规范:** `markdownlint-cli` (用于 `Makefile` 中的 `lint` 任务)
*   **版本控制:** Git
*   **自动化:** Makefile
*   **操作系统:** 兼容 Linux

## 主要功能与工作流程 (Key Features & Workflow)

1.  **AI 提示词库 (`i18n/zh/prompts/`):**
    *   一个极其庞大和精细分类的提示词集合，是项目的核心资产。
    *   `coding_prompts/`: 专注于编程和代码生成的提示词。
    *   `system_prompts/`: 用于设定 AI 行为和思维框架的系统级提示词。
    *   `user_prompts/`: 用户自定义或常用的提示词。
    *   `meta_prompts/`: 元提示词与提示工程辅助。

2.  **提示词库管理工具 (`libs/external/prompts-library/`):**
    *   提供 Python 工具 (`main.py`)，用于在 Excel 工作簿 (`prompt_excel/`) 和 Markdown 文档 (`prompt_docs/`) 之间进行提示词的相互转换。
    *   支持交互式和非交互式操作。

3.  **技能库 (`i18n/zh/skills/`):**
    *   一个模块化的技能集合，为 AI 提供了特定工具和领域的知识。
    *   每个技能（如 `ccxt`, `postgresql`, `telegram-dev`）都包含独立的 `SKILL.md` 描述, 参考资料和脚本。

4.  **项目备份工具 (`backups/`):**
    *   `快速备份.py` 脚本能根据 `.gitignore` 规则智能地打包项目文件为 `.tar.gz` 格式。

5.  **知识库与文档 (`i18n/zh/documents/`):**
    *   包含代码组织、开发经验、系统提示词构建原则、项目架构模板等各类文档。

6.  **外部工具与个人配置 (`libs/external/`):**
    *   存放非核心项目代码但有用的外部工具、个人配置或实验性代码。例如：`my-nvim/` (nvim 配置), `XHS-image-to-PDF-conversion/` (图片转PDF工具)。

## 文件结构 (File Structure)

```
.
├── .gitignore                   # Git 版本控制忽略文件配置
├── AGENTS.md                    # 面向 AI Agent 的贡献与行为准则。
├── CLAUDE.md                    # 面向 Claude 模型的上下文与指令。
├── CODE_OF_CONDUCT.md           # 项目行为准则。
├── CONTRIBUTING.md              # 贡献指南。
├── GEMINI.md                    # 面向 Gemini 模型的上下文与指令 (本文档)。
├── LICENSE                      # 项目许可证。
├── Makefile                     # 项目自动化脚本 (lint, backup 等)。
├── README.md                    # 项目主文档，包含项目概览、使用指南等。
│
├── i18n/
│   ├── zh/{documents,prompts,skills}   # 中文主语料与方法论。
│   ├── en/{documents,prompts,skills}   # 英文版本资产。
│   ├── he/{documents,prompts,skills}   # 希伯来语（以色列）。
│   ├── es|hi|ar|pt|ru|fr|de|ja|ko|it|tr|nl|pl|id|vi|th|fa|uk|bn|ta|ur|ms|sw|ha/{documents,prompts,skills}  # 其他常用语言骨架。
│
├── libs/                        # 核心库代码。
│   ├── common/                  # 通用功能和工具库。
│   │   ├── __init__.py          # Python 包初始化文件。
│   │   ├── models/              # 数据模型定义。
│   │   └── utils/               # 实用工具函数。
│   ├── database/                # 数据库相关代码。
│   └── external/                # 外部工具、个人配置或实验性代码。
│       ├── prompts-library/     # 提示词库管理工具 (Excel-Markdown 互转)。
│       │   ├── main.py          # 提示词库管理工具主程序。
│       │   ├── requirements.txt # 工具依赖。
│       │   ├── prompt_excel/    # Excel 格式提示词。
│       │   ├── prompt_docs/     # Markdown 格式提示词文档。
│       │   └── ... (其他 prompts-library 内部文件)
│       ├── l10n-tool/           # 多语言批量翻译脚本，保护代码块，先机翻后润色。
│       ├── my-nvim/             # 个人 Neovim 配置。
│       └── XHS-image-to-PDF-conversion/ # 小红书图片转 PDF 工具。
│
├── i18n/zh/prompts/                     # 核心资产：AI 提示词库。
│   ├── coding_prompts/                  # 编程与代码生成相关提示词。
│   ├── system_prompts/                  # AI 系统级提示词（含 CLAUDE 版本目录）。
│   ├── user_prompts/                    # 用户自定义提示词。
│   └── meta_prompts/                    # 元提示词与提示工程辅助。
│
└── i18n/zh/skills/                      # 模块化技能库。
    ├── ccxt/                    # CCXT 加密货币交易库技能。
    ├── claude-code-guide/       # Claude Code 使用指南技能。
    ├── postgresql/              # PostgreSQL 数据库技能。
    ├── telegram-dev/            # Telegram Bot 开发技能。
    └── ... (其他 10+ 个技能)
```

---
> Source: [2025Emma/vibe-coding-cn](https://github.com/2025Emma/vibe-coding-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
