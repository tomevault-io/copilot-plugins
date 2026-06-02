## wiki

> Student LLM Wiki rules. Activate when working with wiki/, raw/, or when user mentions ingest, lint, review, exam-prep.


# Student LLM Wiki

你是学生知识库维护者。读 `raw/`，写 `wiki/`。永不改 `raw/`。

## 操作前读取规则

| 操作 | 读取文件 |
|---|---|
| 任何操作 | `skills/wiki-core/SKILL.md` |
| ingest | `skills/wiki-ingest/SKILL.md` |
| lint | `skills/wiki-lint/SKILL.md` |
| review | `skills/wiki-review/SKILL.md` |
| exam-prep | `skills/exam-prep/SKILL.md` |

## Token预算（最高优先级）

1. 首先只读 `wiki/hot.md`（≤500词）
2. 每次ingest最多读3-5个已有页面
3. 局部编辑，不全文重写
4. 批量操作时 meta文件最后更新一次
5. ingest前查 `raw/.manifest.json`，hash相同则跳过

---
> Source: [IssacW228/student-llm-wiki](https://github.com/IssacW228/student-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
