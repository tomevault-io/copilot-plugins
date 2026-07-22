## personalized-interactive-learning

> 这是一套基于 Claude Code（兼容 Codex）的交互式学习 skill 套件。5 个 skill 覆盖从了解自己、研究领域、学习课程、深度阅读到综合咨询的完整学习闭环。

# 定制化交互式学习系统

这是一套基于 Claude Code（兼容 Codex）的交互式学习 skill 套件。5 个 skill 覆盖从了解自己、研究领域、学习课程、深度阅读到综合咨询的完整学习闭环。

---

## 文件结构

```
vault/
├── CLAUDE.md                              # 你正在读的这个文件，系统总纲
├── skills/                                # 技能层（5 个自定义 skill）
│   ├── life-interviewer/                  # 人生采访师 — 了解用户 + 沉淀档案
│   │   ├── SKILL.md
│   │   └── references/
│   ├── industry-research/                 # 行业研究 — 系统性研究陌生领域
│   │   ├── SKILL.md
│   │   └── references/
│   ├── course-learning/                   # 课程学习 — 基于固定资料学完一门课
│   │   ├── SKILL.md
│   │   └── references/
│   ├── book-reading/                      # 深度阅读 — 逐部分读透一本书
│   │   ├── SKILL.md
│   │   ├── references/
│   │   └── scripts/
│   └── knowledge-consultation/            # 知识咨询 — 基于已学知识提供个性化建议
│       └── SKILL.md
├── wiki/                                  # 知识沉淀层（skill 产出）
│   ├── user_wiki/                         # 用户个人档案（7 个维度）
│   ├── book_wiki/                         # 书籍阅读记录
│   ├── research_wiki/                     # 行业/概念研究成果
│   └── course_wiki/                       # 课程学习记录
└── raw/                                   # 原材料层（用户放入）
    ├── user_raw/                          # 用户个人原始素材
    └── book_raw/                          # 书籍 PDF 文件
```

---

## 五个 Skill 总览

### 1. life-interviewer — 人生采访师

**做什么**：通过 5 阶段时间线回溯（童年 → 学生 → 大学 → 工作 → 当下），完整了解用户人生经历，将信息沉淀到 7 维度用户档案。

**产出**：`wiki/user_wiki/` 下的结构化个人档案

**触发方式**：
- "我想聊聊我的经历"
- "帮我梳理一下我的人生"
- "帮我沉淀一下刚才聊的"

### 2. industry-research — 行业研究

**做什么**：系统性地帮用户研究任何不熟悉的行业、概念或领域。AI 自行搜集资料（T1-T4 可信度分级），通过 MISSION 驱动的教学循环帮用户建立结构化认知。

**产出**：`wiki/research_wiki/{主题}/` 下的 MISSION + PROGRESS + RESOURCES + 课程文件

**触发方式**：
- "帮我研究一下 XX 行业"
- "XX 是什么？帮我科普一下"
- "分析一下 XX"

**特色**：T1-T4 资料来源分级、🟢🟡🔴 三层理解评估、读 user_wiki 定制化提问

### 3. course-learning — 课程学习

**做什么**：基于用户提供的固定学习资料（课程链接、逐字稿、视频字幕等），系统性帮用户完成一门课程。自动提取 YouTube/Bilibili/网页内容，制定 CURRICULUM 学习地图，逐节教学。

**产出**：`wiki/course_wiki/{课程名}/` 下的 MISSION + CURRICULUM + PROGRESS + 课程文件

**触发方式**：
- "帮我学这门课"
- "这是课程链接，帮我学一下"
- "帮我提取课程内容"

**特色**：内置 YouTube/Bilibili/网页提取工具、保留讲师原话、🟢🟡🔴 评估

### 4. book-reading — 深度阅读

**做什么**：辅助用户深度阅读一本书。提取 PDF 目录 → 明确阅读目的 → 生成阅读蓝图（3-5 部分）→ 逐部分苏格拉底诘问对话。

**产出**：`wiki/book_wiki/{书名}/` 下的阅读蓝图 + 部分笔记（含诘问记录 + 应用场景）

**触发方式**：
- "帮我读这本书"
- "/read raw/book_raw/书名.pdf"
- 直接粘贴目录 + "开始读《书名》"

**特色**：苏格拉底诘问四阶段（理解→分析→评价→应用）、双重角色（教练→伙伴）、内置 PDF 提取工具

### 5. knowledge-consultation — 知识咨询

**做什么**：结合用户档案和已学知识（书、研究、课程），为用户的具体问题提供个性化、可执行的建议。纯读取型 skill，不创建文件。

**检索范围**：`user_wiki` + `book_wiki` + `research_wiki` + `course_wiki`

**触发方式**：
- "/consult 我想做XX但不知道从哪里开始"
- "帮我分析一下这个问题"
- "基于我学过的知识，给我一些建议"

**特色**：转化而非摘录、约束优先、根据用户思维风格调整建议表达方式

---

## Skill 之间的关系

```
life-interviewer          ← 基础：了解用户是谁
       ↓
   user_wiki  ──────────→ knowledge-consultation  ← 综合检索所有知识
       ↓                           ↑
industry-research  ────────────────┤
       ↓                           ↑
 course-learning  ─────────────────┤
       ↓                           ↑
  book-reading   ──────────────────┘
       ↓
  各自产出到对应的 wiki 子目录
```

- `life-interviewer` 是**基础**——其他 skill 读取 user_wiki 来定制化内容
- `industry-research`、`course-learning`、`book-reading` 是**三个平行的学习 skill**——分别对应研究、课程、书籍三种学习场景
- `knowledge-consultation` 是**上层**——检索所有 wiki，综合输出个性化建议

---

## wiki 目录说明

### user_wiki（用户个人档案）

`life-interviewer` 产出，knowledge-consultation 及其他 skill 读取。

| 维度 | 存什么 |
|------|--------|
| 01_身份与角色 | 职业、社会身份、家庭角色 |
| 02_价值观与信念 | 判断标准、核心信念 |
| 03_底层思维模式 | 认知框架、决策习惯 |
| 04_技能与特长 | 硬技能、软技能、爱好 |
| 05_人际关系网络 | 重要的人、关系性质 |
| 06_人生重大轨迹 | 转折点、关键经历 |
| 07_当下状态与近期事件 | 近期事件、当前目标 |

### book_wiki（书籍阅读记录）

`book-reading` 产出。每本书一个子文件夹，包含阅读蓝图和逐部分笔记（含苏格拉底诘问记录）。

### research_wiki（行业/概念研究）

`industry-research` 产出。每个主题一个子文件夹，包含 MISSION、PROGRESS、RESOURCES 和课程文件。

### course_wiki（课程学习记录）

`course-learning` 产出。每门课一个子文件夹，包含 MISSION、CURRICULUM、PROGRESS 和课程文件。

---

## raw 目录说明

| 目录 | 用途 | 关联 skill |
|------|------|-----------|
| `raw/user_raw/` | 用户个人原始素材（简历、日记、聊天记录等） | life-interviewer |
| `raw/book_raw/` | 书籍 PDF 文件 | book-reading |

**使用方式**：将文件放入对应目录，然后触发关联 skill。

---
> Source: [bintaozhan-hub/personalized-interactive-learning](https://github.com/bintaozhan-hub/personalized-interactive-learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
