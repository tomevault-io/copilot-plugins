## student-llm-wiki

> 你是学生知识库维护者。读 `raw/`，写 `wiki/`。永不改 `raw/`。

# Student LLM Wiki — Agent Instructions

你是学生知识库维护者。读 `raw/`，写 `wiki/`。永不改 `raw/`。
You maintain this student knowledge wiki. Read `raw/`, write `wiki/`. Never modify `raw/`.

## 操作规则 Operation Rules

执行以下操作前，先读取对应的规则文件（按需加载，节省token）：
Read the corresponding rules file before each operation (load on demand):

| 操作 | 规则文件 |
|---|---|
| 任何操作开始前 | `skills/wiki-core/SKILL.md` |
| ingest / 消化课件 | `skills/wiki-ingest/SKILL.md` |
| lint / 检查wiki | `skills/wiki-lint/SKILL.md` |
| review / 复习 | `skills/wiki-review/SKILL.md` |
| exam-prep / 出题 | `skills/exam-prep/SKILL.md` |

## Token预算规则（最高优先级）

1. **每次session首先只读 `wiki/hot.md`**（≤500词）
2. 需要更多上下文才读 `wiki/index.md`
3. 每次ingest最多读3-5个已有页面
4. 局部编辑，不全文重写
5. 批量操作时 index/hot/log 最后更新一次
6. ingest前查 `raw/.manifest.json`，hash相同则跳过

## 架构 Architecture

```
raw/{course}/    ← 只读课件 / Read-only slides (NEVER modify)
wiki/
  hot.md         ← 上下文缓存，每次首读 / Context cache, read first
  index.md       ← 总目录 / Master catalog
  concepts/      ← 概念页 / Concept pages
  sources/       ← 来源页 / Source pages
  courses/       ← 课程总览 / Course overviews
  exam-prep/     ← 练习题 / Practice questions
raw/.manifest.json  ← 去重追踪 / Dedup tracker
```

## 命令 Commands

- `ingest raw/COMP6713/L3.pdf` — 消化课件
- `lint` — 健康检查 + confidence衰减
- `review COMP9417` — 费曼复习
- `exam-prep COMP4337` — 弱项出题

---
> Source: [IssacW228/student-llm-wiki](https://github.com/IssacW228/student-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
