## opencode-novel-plugin

> OpenCode 服务端插件，为 AI 辅助小说创作提供 6 个工具（novel_init、novel_status、novel_save_chapter、novel_context、novel_guide、novel_update）、3 个专属 Agent（novel-planner、novel-writer、novel-editor）、15 个 RTCO 提示词模板（含武侠专属+现实映射）、5 个 Skill（含武侠专项+现实映射），以及完整的文件式数据管理系统。

# opencode-novel-plugin — 小说创作插件

## 概述

OpenCode 服务端插件，为 AI 辅助小说创作提供 6 个工具（novel_init、novel_status、novel_save_chapter、novel_context、novel_guide、novel_update）、3 个专属 Agent（novel-planner、novel-writer、novel-editor）、15 个 RTCO 提示词模板（含武侠专属+现实映射）、5 个 Skill（含武侠专项+现实映射），以及完整的文件式数据管理系统。

> 文档编辑使用 opencode 内置 Edit/Write 工具，`.novel/` 目录下所有文件均可直接编辑。

## 项目结构

```
opencode-noval/
├── src/
│   ├── index.ts              # 插件入口，PluginModule 格式: { id, server }
│   ├── utils.ts              # 路径/文件/解析工具函数 + 类型定义
│   ├── prompts.ts            # 15 个 RTCO 提示词模板（含武侠专属+现实映射）
│   ├── templates.ts          # 11 个文档模板 + 填充函数
│   ├── agents/
│   │   ├── novel-planner.ts  # novel-planner 策划师 agent
│   │   ├── novel-writer.ts   # novel-writer 作者 agent
│   │   └── novel-editor.ts   # novel-editor 编辑 agent
│   └── tools/
│       ├── init.ts           # novel_init — 项目初始化
│       ├── status.ts         # novel_status — 状态报告
│       ├── save.ts           # novel_save_chapter 章节保存
│       ├── context.ts        # novel_context — 写作上下文构建
│       ├── guide.ts          # novel_guide — 工作流指南
│       └── track.ts          # novel_update — 状态追踪更新
├── commands/                # 斜杠命令
│   ├── novel.md          # /novel — 智能入口（自动检测阶段）
│   ├── novel-init.md     # /novel-init — 初始化项目
│   ├── novel-concept.md  # /novel-concept — 故事概念设计
│   ├── novel-world.md    # /novel-world — 世界观构建
│   ├── novel-characters.md # /novel-characters — 角色设计
│   ├── novel-outline.md  # /novel-outline — 大纲编写
│   ├── novel-scenes.md   # /novel-scenes — 场景设计
│   ├── novel-reality.md  # /novel-reality — 现实映射素材发掘
│   ├── write-chapter.md  # /write-chapter — 写章命令
│   ├── novel-status.md   # /novel-status — 状态查看命令
│   └── novel-review.md   # /novel-review — 审稿命令
├── skills/                  # 创作技能
│   ├── novel-workflow/   # 完整6阶段工作流 skill
│   ├── novel-writing/    # 章节写作 skill（反AI规则+感官指南）
│   ├── novel-editor/     # 审稿修订 skill（一致性+六稿修改法）
│   ├── novel-wuxia/      # 武侠专项 skill（功法/门派/战斗/桥段库）
│   └── novel-reality/    # 现实映射 skill（批判现实主义/陌生化/原型提取）
├── package.json
└── tsconfig.json
```

## 数据目录（.novel/）

```
.novel/
├── config.json              # 项目配置（标题/类型/字数/阶段）
├── concept.md               # 故事概念
├── world-building.md        # 世界观设定
├── characters/
│   ├── profiles.md          # 角色总览（清单/关系图/冲突矩阵）
│   ├── {角色名}.md           # 每个角色独立档案（驱动力三角/弧线/章节状态）
│   └── state.json           # 角色状态追踪
├── outline-brief.md         # 简要大纲
├── outline-detailed.md      # 详细大纲（或拆分为 outline/chapter-001.md 等）
├── scenes.md                # 场景列表（或拆分为 scenes/chapter-001.md 等）
├── summary.md               # 前文摘要（每章更新）
├── foreshadow.json          # 伏笔追踪
├── wuxia/                   # 武侠专属（genre=武侠时自动创建）
│   ├── sects.md             # 门派势力（或拆分为 sects/{门派名}.md）
│   ├── martial-arts.md      # 功法谱
│   └── weapons.md           # 兵器谱（或拆分为 weapons/{兵器名}.md）
├── reality-source.md        # 现实映射素材库（或拆分为 reality/events/ 等子目录）
└── chapters/                # 章节正文
    ├── chapter-001.md
    ├── chapter-002.md
    └── ...
```

## 工具清单

| 工具 | 用途 | 关键参数 |
|------|------|---------|
| `novel_init` | 初始化项目 | title, genre, targetWords, chapterCount |
| `novel_status` | 查看进度 | detailed |
| `novel_save_chapter` | 保存章节 | chapterNumber, title, content |
| `novel_context` | 构建写作上下文 | chapterNumber, includePrompts |
| `novel_guide` | 工作流指南 | section |
| `novel_update` | 更新追踪 | action, content, chapterNumber |

> 文档编辑（概念/世界观/角色/大纲/场景等）使用 opencode 内置 Edit/Write 工具，路径映射见 Agent prompt。

## Agent 清单

| Agent | 用途 | 模式 | 温度 |
|-------|------|------|------|
| `novel-planner` | 故事策划（概念/世界观/角色/大纲/场景）+ 现实素材发掘 | subagent | 0.3 |
| `novel-writer` | 章节正文创作 | subagent | 0.7 |
| `novel-editor` | 审稿修订（一致性检查/风格润色/节奏分析） | subagent | 0.1 |

## 技能清单

| Skill | 用途 | 加载时机 |
|-------|------|---------|
| `novel-workflow` | 6阶段完整工作流 | 规划阶段 |
| `novel-writing` | 章节写作指导 | 写正文时 |
| `novel-editor` | 审稿修订 | 审稿时 |
| `novel-wuxia` | 武侠专项（功法/门派/战斗/桥段库） | 武侠创作时 |
| `novel-reality` | 现实映射（批判现实主义/陌生化技法/原型提取/合规指南） | 现实映射时 |

## 命令清单

| 命令 | 用途 | 参数 |
|------|------|------|
| `/novel` | 自动检测当前阶段并引导下一步 | 无 |
| `/novel-init` | 初始化新小说项目 | [书名] |
| `/novel-concept` | 设计故事核心概念 | 无 |
| `/novel-world` | 构建世界观设定 | 无 |
| `/novel-characters` | 设计角色群 | 无 |
| `/novel-outline` | 编写故事大纲 | 无 |
| `/novel-scenes` | 场景设计 | 无 |
| `/novel-reality` | 现实映射素材发掘 | 无 |
| `/write-chapter` | 写一章正文 | 章节编号 |
| `/novel-status` | 查看进度 | 无 |
| `/novel-review` | 审稿 | 章节编号/all, --focus |

## 工作流阶段

```
0:概念 → 1:世界观 → 2:角色 → 3:大纲 → 4:场景 → 5:正文(循环)
```

每个阶段有明确的输入、输出和检查点。前一阶段完成是下一阶段的前提。

## 提示词模板

11+2+2个RTCO框架提示词，覆盖从概念到审稿的全部阶段：
concept, world-building, characters, outline-brief, outline-detailed, scenes, chapter-write, chapter-continue, consistency-check, character-state-update, summary-update

武侠专属: wuxia-martial-arts, wuxia-combat

现实映射专属: reality-source, character-archetype

## 约定

- 所有数据文件使用 UTF-8 编码
- Markdown 文档用于人类可读内容（概念、世界观、角色等）
- JSON 文件用于程序化数据（配置、状态、伏笔）
- 章节文件自动编号（chapter-001.md, chapter-002.md, ...）
- 字数统计：中文字符 + 英文单词
- 项目配置存储在 `.novel/config.json`

## 依赖

- `@opencode-ai/plugin` >= 1.4.0（peer dependency）
- `zod` ^3.24.0（参数验证）

---
> Source: [wmzy/opencode-novel-plugin](https://github.com/wmzy/opencode-novel-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
