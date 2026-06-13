## academic-paper-workflow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这个仓库是什么

**不是应用程序，是中文学术论文 AI 辅助写作的 skill 框架。** "代码"由四部分组成：

- `.claude/skills/paper-workflow/SKILL.md` — 10 阶段工作流定义，作为 `/paper-workflow` 斜杠命令暴露给论文作者
- `.claude/skills/paper-workflow/scripts/*.py` — init / 构建 / 校验脚本（被 SKILL.md 调用）
- `.claude/skills/paper-review-team/` — Stage 7 的多 agent 评审 skill（独立可调用）
- `templates/{journal-paper,thesis,essay}/` — 论文项目模板
- `references/*.md` — 写作知识库（头脑风暴、检索实操、各节写作语步、评审、AI 模式检测）

**两种视角**：终端用户通过 `/paper-workflow <stage>` 写论文（用法见 `README.md`）；在此仓库工作 = 维护框架本身（改 SKILL.md / 脚本 / 模板 / 知识库）。

## 架构（关键设计）

- **Markdown 是唯一源文件**：论文写为 `draft/paper.md`，经脚本转 docx。格式与内容分离，diff 可读。
- **引用而非复制**：工作流各阶段委托给**外部** skill（`deep-research`/`academic-paper`/`academic-paper-reviewer`/`md-to-docx`/`humanizer-cn`，经 42plugin 安装）。本仓库只提供编排 + 两个内置 skill。改某阶段行为前先确认它是否实际跑在外部 skill 里。
- **单一信息源**：`paper-review-team` 的评审方法论引用 `paper-workflow/references/peer-review-simulation.md`，自身不复制。改评审逻辑时两处都要看。
- **四层元架构**（认知边界/意图防护/执行可靠/持续优化）是 SKILL.md 内对工作流阶段的约束，不是运行时代码——别去找它的实现。

## 命令

### 开发（维护本框架时）

```bash
uv sync                    # 安装依赖（python-docx, matplotlib, pillow；dev: ruff）
uv run ruff check .        # lint
uv run ruff format .       # 格式化

# 跑单个脚本：脚本自身目录会自动进 sys.path，裸 import _academic_db / validate_citations 可用
uv run python .claude/skills/paper-workflow/scripts/init_project.py --name "测试标题" -t journal-paper -o ./_scratch
uv run python .claude/skills/paper-workflow/scripts/validate_citations.py --input <paper>/draft/paper.md
uv run python .claude/skills/paper-workflow/scripts/verify_references.py --input <paper>/draft/paper.md --offline   # 仅格式校验，离线
uv run python .claude/skills/paper-workflow/scripts/verify_references.py --input <paper>/draft/paper.md             # 含在线存在性验证

# PPT 编译（可选，独立 Node 工具链）
cd templates/journal-paper/slides && npm install && node compile.js
```

**无单元测试套件**。脚本的验证方式 = 在一篇示例论文上跑 `doctor` / `validate` / `verify` 看输出正确。改脚本时带上一个真实/示例 `paper.md` 做冒烟测试。

### 工作流（终端用户，在 Claude Code 内）

`/paper-workflow <stage>`，10 阶段：`init → brainstorm → research → outline → write → figures → review → revise → build → archive`。辅助命令：`status`、`doctor`（依赖检查）、`validate`、`verify`、`build-ppt`、`watchdog`。`/paper-review-team` = Stage 7 多 agent 增强执行器。**完整命令表与参数见 `SKILL.md`，勿在此重复。**

## 关键约定

- **`validate` ≠ `verify`**：`validate`（`validate_citations.py`）查引文**编号**一致性，快、离线，build 前置；`verify`（`verify_references.py`）查参考文献**真实性**（GB/T 7714 格式合规 + 数据库存在性），检测 AI 编造，评审阶段用。`verify_references.py` 内部复用 `validate_citations.validate`。
- **脚本共享模块**：`_academic_db.py`（OpenAlex/Crossref/S2 查询、题名相似度、GB/T 7714 归一化）被 `verify_references.py` 等裸 import 复用。新校验脚本沿用同一模块，勿重复实现数据库查询。
- **扩展点**：新论文类型 → `templates/`；新写作语步 → `references/section-rhetorical-moves/`；新构建/校验脚本 → `scripts/` 并在 `SKILL.md` 命令表注册；新阶段委托 → `SKILL.md` 技能集成表加行。

## 规范

- git commit message 用**中文**（项目级硬性要求）。
- Python 包管理用 `uv`，禁止 pip/poetry，项目内 `.venv`。
- 媒体文件（mp3/png/docx/pptx 等）已在 `.gitignore` 排除，构建产物 `output/` 同理——勿提交。

---
> Source: [Joe-rq/academic-paper-workflow](https://github.com/Joe-rq/academic-paper-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
