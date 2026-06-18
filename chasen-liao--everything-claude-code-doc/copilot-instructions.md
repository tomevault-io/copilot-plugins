## everything-claude-code-doc

> 本文件给 AI 助手阅读使用, 不是给人读的项目文档。如需了解仓库背景, 看 `README.md`; 如需了解维护约定, 看 `CLAUDE.md`。

# Repository Guidelines

本文件给 AI 助手阅读使用, 不是给人读的项目文档。如需了解仓库背景, 看 `README.md`; 如需了解维护约定, 看 `CLAUDE.md`。

## Project Overview

`Everything-claude-code-Doc` 是 [`affaan-m/Everything Claude Code (ECC)`](https://github.com/affaan-m/ECC) 的**中文教程镜像仓库**。它把上游 ECC 的概念、命令、阅读路径翻译/梳理为 GitHub Pages 站点, 同时通过嵌入式 gitlink 保留上游的完整原文以供对照。

权威解释始终以**上游为准**; 本仓库"教程层"只翻译、串联、给出阅读路径, 不复制完整原始文档。

## Architecture & Data Flow

仓库由三层组成, 数据方向明确:

```
┌────────────────────────────────────────────────────────────┐
│ 顶层 zh 教程门牌  (本仓库维护, 不进 Pages build)           │
│   README.md / ECC-COMMANDS-ZH.md / ECC-NAVIGATION-ZH.md    │
└────────────────────────────────────────────────────────────┘
                          │ scripts/migrate_docs.py
                          │ 一次性拆分 + 链接改写 (已完成)
                          ▼
┌────────────────────────────────────────────────────────────┐
│ docs/  Pages 站点源  (本仓库维护)                          │
│   getting-started/  concepts/  commands/  routes/          │
│   upstream-navigation/  maintenance/  faq.md  index.md     │
│   → MkDocs Material 渲染 → GitHub Pages                    │
└────────────────────────────────────────────────────────────┘
                          │ 链接指向
                          ▼
┌────────────────────────────────────────────────────────────┐
│ everything-claude-code/  上游 gitlink  (本仓库不编辑)      │
│   mode 160000, 无 .gitmodules, fast-forward 同步            │
│   → mkdocs.yml 的 exclude_docs 显式排除, 不进 build         │
└────────────────────────────────────────────────────────────┘
```

辅助层: `scripts/migrate_docs.py` 把"顶层三份 zh 文档"一次性拆分/链接改写到 `docs/`。首版已完成, 默认不重跑。

## Key Directories

| 路径 | 用途 | 备注 |
| --- | --- | --- |
| `docs/` | Pages 站点源 | 19 个 `.md` + 1 个 CSS, 无 frontmatter |
| `docs/getting-started/` | 入门三页(是什么/适合谁/快速开始) | 入口 |
| `docs/concepts/` | 核心概念(总览/三组件/常见误解) | 最长的 `agent-vs-skill-vs-command.md` 在此 |
| `docs/commands/` | 命令速查(总览/最常用/按场景/按语言/工作流) | 大量 GFM 表格 + `text` 代码块 |
| `docs/routes/` | 推荐阅读路线 | 多个 H3 路线分支 |
| `docs/upstream-navigation/` | 上游副本导航 | 帮助读者定位上游目录 |
| `docs/maintenance/` | 维护页(与上游同步) | 留有 TODO 占位 |
| `docs/assets/extra.css` | 站点样式占位 | 当前仅一行 CSS 注释 |
| `docs/superpowers/{plans,specs}/` | Agent 计划/规格 | **不进 build** |
| `scripts/` | 一次性迁移脚本 | 仅有 `migrate_docs.py` |
| `tests/` | 迁移脚本的 pytest 套件 | 16 个测试 |
| `everything-claude-code/` | **上游 gitlink** | 不要编辑, 只 ff 同步 |
| `.claude/skills/sync-upstream-zh-docs/` | 维护 skill | 同步流程在此 |

## Development Commands

```bash
# 依赖安装
pip install -r requirements.txt

# 本地预览 (默认 http://127.0.0.1:8000)
python -m mkdocs serve

# 严格构建 (用于本地/CI 验证, 任何 WARNING 都会 fail)
python -m mkdocs build --strict

# 跑迁移脚本测试 (16 个)
python -m pytest tests/

# 一次性迁移脚本 (默认不重跑, 仅作 dry-run 预览)
python -m scripts.migrate_docs --dry-run

# 同步上游 (在子仓库内)
cd everything-claude-code
git fetch origin
git rev-list --left-right --count HEAD...origin/main
git merge --ff-only origin/main
cd ..
git add everything-claude-code   # 写回新 SHA 到主仓库
```

## Code Conventions & Common Patterns

### Markdown 写作 (`docs/*.md`)

- 无 YAML frontmatter; 无 admonition/tabbed/details 块; 无 emoji 或 Material 图标(刻意保持极简)。
- H1/H2/H3 标准层级。**常见模式是 H1 标题后第一个 H2 与 H1 同名**——不要"修正"这种重复, 是项目有意为之的样式。
- 命令清单一律用 GFM 表格 + `text` 围栏代码块展示 slash 命令。
- 链接二选一, 互斥:
  - 站内相对路径: `[text](../commands/index.md)`
  - 上游 GitHub URL: `[text](https://github.com/affaan-m/ECC/blob/main/...)`
- **禁止**在 `docs/` 下使用 `everything-claude-code/...` 形态的相对路径。

### Python (`scripts/migrate_docs.py`)

- 单文件, ~254 行, 一个 argparse CLI。
- 公开 API: 常量 `REPO_ROOT`, `UPSTREAM_BASE`, `LINK_RULES`, `MAPPING`, `SOURCE_FILES`; 函数 `rewrite_links`, `parse_h2_sections`, `build_target_files`, `main`。
- 链接改写顺序敏感: `LINK_RULES` 是 6 条 `(re.Pattern, replacer)` 列表, **目录规则(末尾 `/`)必须排在非 md 文件规则之前**——见文件内注释 `Order matters: ...`。
- I/O 用 `pathlib.Path.write_text(..., encoding="utf-8")`; 状态输出用 `print`, 不引 `logging`。
- 类型注解用 Python 3.9+ 风格(`list[...]`, `dict[...]`), 配合 `from __future__ import annotations`。
- 入口: `python -m scripts.migrate_docs [--dry-run]`; 不自动 commit。

### 命名 / 文件组织

- 子包通过 `__init__.py` 单行 `# package marker` 注释即可; 无 re-export。
- 测试模块命名 `test_*.py`; 测试函数命名 `test_xxx_yyy`; 不分类、不参数化、不夹具——保持可读。

### 状态管理

- 仓库无运行时状态。`site/` 是 MkDocs build 产物, 已加入 `.gitignore`。
- `.pytest_cache/` 是 pytest 自动生成, 不入库。

## Important Files

| 文件 | 角色 |
| --- | --- |
| `mkdocs.yml` | MkDocs 站点配置; `exclude_docs` 显式排除 `superpowers/**` 与 `everything-claude-code/**`。`nav:` 与 `docs/` 树必须保持一一对应。 |
| `requirements.txt` | 2 行: `mkdocs-material>=9.5`, `mkdocs-material-extensions>=1.3`。MkDocs 本身是传递依赖。 |
| `.github/workflows/deploy-pages.yml` | 唯一 CI: push 到 main 且触及 `docs/**`/`mkdocs.yml`/`requirements.txt`/自身时触发; Python 3.12 + `mkdocs build --strict` + `actions/deploy-pages@v4`。 |
| `.gitignore` | 10 行: `site/`, `__pycache__/`, `*.pyc`, `.pytest_cache/`, `.venv/`, `venv/`, `.DS_Store`, `Thumbs.db`。 |
| `scripts/migrate_docs.py` | 一次性迁移 CLI(见上)。 |
| `tests/test_migrate_docs.py` | 16 个 pytest 测试, 无 `conftest.py` / `pytest.ini`。 |
| `CLAUDE.md` | 项目级 Claude memory; 已写明"gitlink 写回、59 vs 79、CRLF 警告"等易错点。 |
| `.claude/skills/sync-upstream-zh-docs/SKILL.md` | 维护 skill; 用户说"同步上游 / 刷新中文文档"时**先调用此 skill**。 |
| `LICENSE` | 保留上游许可原文, 不修改。 |

## Runtime/Tooling Preferences

- **Python 3.12** (CI 固定; 本地 >= 3.9 即可满足类型注解风格)。
- **包管理**: 仅 `pip install -r requirements.txt`, 无 Pipfile/poetry/uv lockfile, 无 `.python-version`。
- **唯一 CI**: `.github/workflows/deploy-pages.yml`, 用 `actions/setup-python@v5` + `cache: pip`, 部署动作 `actions/deploy-pages@v4`。
- **Git 策略**: 嵌入式 gitlink 只能 `--ff-only`; 不要 rebase 或非 ff merge。提交时主仓库要 `git add everything-claude-code` 把新 SHA 写回。
- **平台**: Windows 友好; CRLF 警告是 `core.autocrlf` 的正常表现, 不要改文件换行。
- **不要新建 CI workflow**: 当前 Pages 部署是唯一的 CI; 测试本地跑。

## Testing & QA

- **测试框架**: pytest, 路径通过 `from scripts.migrate_docs import ...` 直接导入, 依赖 `scripts/__init__.py` 和 `tests/__init__.py` 把目录注册为包。
- **运行**: `python -m pytest tests/` (16 个测试, 全部 0.0x 秒级)。
- **覆盖范围**:
  - `rewrite_links` × 9: md 文件 → blob, 目录 → tree, 脚本文件 → blob, 三个顶层 .md 互跳, 保留 https, 同行多链接, 根目录。
  - `parse_h2_sections` × 4: 三个 H2 拆分, 保留正文, 剥离 HR, 忽略 H1+intro。
  - `build_target_files` × 3: 多 H2 合并, 正文内链接改写, H2 顺序遵循 MAPPING。
- **MkDocs 验证**: 写完 `docs/` 必须 `python -m mkdocs build --strict` 通过(任何未链接/导航漂移/WARNING 都会 fail)。
- **CI 不跑 pytest**; 修改 `scripts/migrate_docs.py` 后请本地补跑, 失败要修复而不是放宽断言。

## Don'ts (高频错误)

- **不要编辑 `everything-claude-code/` 下任何文件**; 同步只能 fast-forward。
- **不要在 `docs/*.md` 里使用 `everything-claude-code/...` 路径**; 用 `https://github.com/affaan-m/ECC/...`。
- **不要混用 "59" 和 "79"**: 79 是 `commands/` 目录文件数(含 `legacy-command-shims/`), 59 是 `COMMANDS-QUICK-REF.md` 列的全局 slash command 数。
- **不要把"63 agents / 249 skills / 79 legacy command shims"与"59 slash commands"混为一谈**: 前者是上游 README 公开承诺的同步面, 后者是全局安装数。
- **不要重跑 `python -m scripts.migrate_docs`**(首版已完成); 后续编辑直接改 `docs/`。
- **不要修改 `LICENSE` 或 `everything-claude-code/README.zh-CN.md`**(上游维护)。
- **不要用 `git rebase` / 非 ff merge** 推进 `everything-claude-code/`; diff 不可读。
- **不要"修正"`docs/` 下"H1 与 H2 同名"的重复**; 是项目有意为之的样式。
- **不要在仓库新建第二个 CI workflow**; Pages 部署是唯一的 CI。
- **不要为通过测试而压测/删断言/放宽断言**。

---
> Source: [Chasen-Liao/Everything-claude-code-Doc](https://github.com/Chasen-Liao/Everything-claude-code-Doc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
