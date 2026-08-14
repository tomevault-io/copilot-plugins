## binance-life

> │       ├── AGENTS.md

# 币安人生仓库说明

## 目录结构

```text
.
├── .gitattributes
├── AGENTS.md
├── README.md
├── assets/
│   └── tasks/
│       ├── AGENTS.md
│       ├── lessons.md
│       └── todo.md
├── raw/
│   ├── README.md
│   ├── 内容提炼.md
│   ├── 币安人生 - 赵长鹏.epub
│   ├── build_epub_币安人生_带图_render/
│   └── cz-skill-main/
└── wiki/
    ├── AGENTS.md
    ├── README.md
    ├── bases/
    ├── dashboard.md
    ├── index.md
    ├── log.md
    ├── maps/
    ├── .obsidian/
    ├── overview.md
    ├── entities/
    ├── sources/
    ├── timelines/
    └── topics/
```

## 层级职责

- `raw/`：原始资料层。默认不可修改，只作为引用与再整理的输入。
- `wiki/`：知识层。所有结构化总结、实体页、主题页、时间线、查询沉淀都放这里。
- `assets/tasks/`：过程层。记录当前任务状态、复盘和经验，避免上下文只留在对话里。
- `.gitattributes`：定义 GitHub 提交时的文本归一化、二进制类型和派生文件策略。

## 仓库约束

- 不要直接修改 `raw/` 中的原始材料。
- 每次 ingest 新资料，至少同步更新 `wiki/index.md` 与 `wiki/log.md`。
- 涉及监管、刑责、法律争议的表述，默认标注为“作者自述”或“待外部验证”，不要把回忆录口径直接写成定论。
- 页面命名保持英文文件名，页面标题与正文说明使用中文。
- 新增结构性目录时，必须同步补齐对应 `AGENTS.md`。
- `wiki/.obsidian/` 下的配置用于这个知识库的原生体验层，当前只共享 `graph.json`、`appearance.json`、`core-plugins.json` 与 `snippets/`，不要把 `workspace.json` 一类个人状态文件混进版本库。
- 如果新增大的二进制或派生文件，记得同步更新 `.gitattributes`，不要把 GitHub diff 面板污染成噪音。

## 默认工作流

1. 读 `wiki/index.md` 获取现有地图。
2. 读相关来源页确认资料边界。
3. 更新实体页、主题页、时间线页。
4. 回写 `wiki/index.md` 与 `wiki/log.md`。
5. 在 `assets/tasks/todo.md` 记录状态，在需要时把教训写入 `assets/tasks/lessons.md`。

## 变更记录

- `2026-04-11`：初始化“币安人生” LLM Wiki 仓库、首轮页面结构与版本控制基线。
- `2026-04-11`：补充 Obsidian 工作台，并开始纳入 `raw/cz-skill-main/` 的二轮研究结论。
- `2026-04-11`：新增 `wiki/maps/` 关系图谱目录，用于沉淀可浏览的实体与主题关系图。
- `2026-04-11`：补充锚点实体与 Obsidian Graph View 配置，提升关系图谱颗粒度。
- `2026-04-11`：引入 Obsidian 原生 `Bases`、`CSS snippets` 与专业化中控台布局，提升浏览体验与美观度。
- `2026-04-11`：补充 GitHub 友好的目录说明与 `.gitattributes`，为正式推送做整理。

---
> Source: [tradecatlabs/Binance-Life](https://github.com/tradecatlabs/Binance-Life) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
