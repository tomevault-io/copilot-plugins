## political-signal-analysis

> - 本项目的核心是“政治信号分析”，不是通用新闻摘要或热榜聚合。

# 项目约定

- 本项目的核心是“政治信号分析”，不是通用新闻摘要或热榜聚合。
- 第一阶段先完成并校准 `skills/political-signal-analysis`，在评测通过前不搭建 SaaS 外壳。
- 所有输出必须区分可核实事实、分析推断和待验证假设。
- 先判断一条信息承担了什么政治功能，再判断发声者是否被协调或直接受命；不能把二者混为一谈。
- 每条分析必须双向扫描：正向任务和负向风险同时判断，尤其检查利好背后暴露的压力、被排除对象和代价承担者。
- 默认使用中文；未经明确要求，不自动交易、不代替用户作出投资决策。
- `skills/political-signal-analysis` 是编写与校准源；`.agents/skills/political-signal-analysis` 是供 Codex 自动发现的仓库级运行副本。修改源 Skill 后必须同步运行副本，并执行 `scripts/verify-migration.ps1` 检查两者一致。
- 对外默认使用 `SSS／SS／S／A／B／C + 机会／风险／混合`；T/C/R/A仅用于内部训练、复盘或用户明确要求时展示。

---
> Source: [klsnslt/political-signal-analysis](https://github.com/klsnslt/political-signal-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
