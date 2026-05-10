## magic-skill

> 本文档定义本 Wiki 的运行方式。每次会话开始时请先阅读本文档。

# LLM Wiki 规范

本文档定义本 Wiki 的运行方式。每次会话开始时请先阅读本文档。

## 项目定位

本项目是**万智牌通用知识基础设施**（Magic: The Gathering General-Purpose Knowledge Infrastructure）。Wiki 是核心知识库，服务于多个消费分支：

- **裁判分支**（referee）—— 精确规则判定、决策树、判例分析
- **策略分支**（strategy）—— 赛制元游戏、套牌分析、组合技拆解（预留）
- **创作分支**（creation）—— 公众号文章、背景故事、可读规则解释（预留）
- **DIY 分支**（diy）—— 卡牌设计规范、关键词设计指南（预留）

**设计原则**：通用知识库（`wiki/concepts/` 等）回答"**这是什么**"（What），各分支专用层回答"**这个分支怎么处理**"（How for this branch）。分支专用层不得破坏通用知识库的内容和结构。

## 目录结构

```
<project-root>/
├── agent/              # Agent 定义（可协作、版本控制）
│   └── mtg-judge-zh.md
├── skill/              # Skill 定义（可协作、版本控制）
│   └── mtg-judge-zh/
│       └── SKILL.md
├── raw/                # 原始资料（文章、论文、图片、数据文件），不可变
│   ├── assets/         # 本地图片和附件
│   ├── cr/             # 完整规则文档（CR）
│   ├── mtr/            # 比赛规则（MTR）
│   ├── ipg/            # 违规处理方针（IPG）
│   ├── references/     # 专题参考文档（待重构为自动生成索引）
│   ├── tools/          # Python 工具集
│   └── data/           # 牌张数据
├── wiki/               # 由 LLM 生成和维护的知识库
│   ├── DESIGN.md       # Wiki 架构设计文档
│   ├── index.md        # 内容索引（每次 Ingest 后更新）
│   ├── log.md          # 按时间顺序追加的操作日志
│   ├── _templates/     # 页面模板
│   ├── sources/        # 每个来源对应一篇摘要页
│   ├── entities/       # 实体页：人物、组织、产品、模型等
│   ├── concepts/       # 概念页：思想、技术、理论、术语
│   ├── synthesis/      # 综合分析：比较、综述、演进中的论点
│   └── branches/       # 各分支专用层
│       ├── referee/    # 裁判分支（当前聚焦）
│       ├── strategy/   # 策略分支（预留）
│       ├── creation/   # 创作分支（预留）
│       └── diy/        # DIY 分支（预留）
└── output/             # 生成的 artifacts（表格、幻灯片、图表、报告）
```

## 命名规范

- 所有 Wiki 页面均为带 YAML frontmatter 的 Markdown 文件。
- 内部链接使用 `[[WikiLink]]` 语法（兼容 Obsidian）。
- 文件名：小写，短横线连接（例如 `attention-is-all-you-need.md`）。
- 中文文件名允许使用，仍需以 `.md` 结尾。
- 每页必须包含以下 frontmatter 字段：
  - `created`：首次创建日期（ISO 格式）
  - `updated`：最近编辑日期（ISO 格式）
  - `type`：`source`、`entity`、`concept`、`synthesis`、`output`、`decision-tree` 之一
  - `tags`：相关标签列表
  - `sources`：本页所依据的原始来源文件名列表（如适用）

## 核心操作

### 1. Ingest（摄入来源）

当用户将文件放入 `raw/` 并说"摄入这个"时：

1. 读取原始文件。
2. 与用户讨论核心要点。
3. 在 `wiki/sources/YYYY-MM-DD-<slug>.md` 创建来源摘要页。
4. 更新或创建相关的 `wiki/entities/` 和 `wiki/concepts/` 页面。
5. 更新 `wiki/index.md`，反映所有新增和修改的页面。
6. 按以下格式在 `wiki/log.md` 追加记录：
   `## [YYYY-MM-DD] ingest | <来源标题>`

### 2. Query（回答问题）

当用户提出问题时：

1. 先读取 `wiki/index.md` 定位相关页面。
2. 读取这些页面并综合出带 `[[引用]]` 的答案。
3. 如果答案内容充实且可复用，询问用户是否将其归档到 `wiki/synthesis/` 或 `output/`。

### 3. Lint（健康检查）

当用户说"检查 Wiki"或类似指令时：

1. 扫描各页面，寻找矛盾、过时主张和缺失的交叉引用。
2. 识别孤立页面（无入链）以及缺少独立页面但重要的术语。
3. 提出具体的修复建议或建议创建的新页面。
4. 若分类发生漂移，更新 `wiki/index.md`。
5. 按以下格式在 `wiki/log.md` 追加记录：
   `## [YYYY-MM-DD] lint | <检查结果的简要描述>`

## 页面模板

创建新页面时，请参考 `wiki/_templates/` 中的对应模板。

---
> Source: [Kuuusoda/magic-skill](https://github.com/Kuuusoda/magic-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
