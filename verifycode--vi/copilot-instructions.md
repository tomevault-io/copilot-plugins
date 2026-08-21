## vi

> 个人投资决策知识库，基于 Karpathy LLM-Wiki 模式：`raw/` 源 → `wiki/` 页面 → 本 schema。

# 投资决策知识库（Investment Decision Wiki）

个人投资决策知识库，基于 Karpathy LLM-Wiki 模式：`raw/` 源 → `wiki/` 页面 → 本 schema。

## 目录约定
- `raw/`：不可变原始资料。LLM 只读，绝不修改，按类型分子目录。
- `wiki/`：LLM 维护的 Markdown 页面。
  - `index.md`：内容索引，每行一页（`- [标题](路径) — 一句话摘要`）
  - `log.md`：时间线日志，追加式，格式 `## [YYYY-MM-DD] 操作 | 说明`
  - `methodology/`：投资体系页面（估值框架、决策 checklist、读书笔记）
  - `entities/stocks/`、`entities/funds/`：个股/基金实体页
  - `topics/`：主题/行业页
  - `summaries/`：源摘要页（含对 raw/ 的引用）

## 命名约定
- 实体页文件名用代码或名称：`600519.md`、`沪深300ETF.md`
- 摘要页：`YYYY-MM-DD-简短标题.md`

## 页面 frontmatter
实体页：

```yaml
---
type: entity-stock | entity-fund
ticker:
name:
sector:
thesis: watch | accumulating | holding | reducing | exit
last_updated: YYYY-MM-DD
---
```

方法论 / 摘要 / 主题页：`type` + `tags` + `last_updated`。

## 实体页正文模板
1. 核心论点（看多/看空，编号列出）
2. 催化剂与风险
3. 估值区间
4. 买入/卖出/退出条件
5. 证据链（引用 summaries/ 或 raw/）
6. 更新历史（追加，最新在最上）

## 反幻觉规则
1. 不确定就标注「无依据」。
2. 永不把未经确认的推断回写为 wiki 页面。
3. 重要写入先给用户看草稿，确认后落盘（nothing durable without review）。

## 工作流（阶段二细化）
- `/ingest`：收录新源
- `/ask`：基于 wiki 提问
- `/lint`：体检

---
> Source: [verifyCode/vi](https://github.com/verifyCode/vi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
