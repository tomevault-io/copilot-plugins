## growth-stock-system

> 基于《教授成长股投资思想》方法论的自动化分析系统，集成基于 Pring《Technical Analysis Explained》(5th Ed) 的技术分析。Python 做数据计算，Claude Code Skills + 多 Agent 做并行分析。

# Growth Stock Analysis System

基于《教授成长股投资思想》方法论的自动化分析系统，集成基于 Pring《Technical Analysis Explained》(5th Ed) 的技术分析。Python 做数据计算，Claude Code Skills + 多 Agent 做并行分析。

## 触发条件

当用户说以下内容时，自动触发主编排器 `growth-orchestrator` skill：

- "分析股票 XX" / "Analyze stock XX"
- "XX 符合成长股框架吗？"
- "帮我评估 XX 的成长潜力"
- "Assess whether XX is a growth stock"
- "跟踪 XX" / "把 XX 加入 Dashboard"

## 目录结构

```
skills/              — 8 个 Claude Code Skill 定义（Skill + 5 Agent + 综合 + 主编排）
scripts/             — Python 数据计算脚本（财务指标、估值、行业对比、技术分析、数据获取）
technical_analysis/  — 技术分析 Python package（指标计算、数据获取、报告生成）
config/              — API Token 配置、跟踪列表、行业映射
data/cache/          — API 响应缓存
data/latest/         — 最新分析中间结果（JSON）
data/reports/        — 最终分析报告（Markdown，含基本面 + 技术面）
tests/               — Python 脚本单元测试
```

## 分析流水线

```
用户指令 → growth-orchestrator（主编排器）
  → Step 1: 确认参数（股票代码、分析深度）+ 🔔 第一次询问小作文/用户想法
  → Step 2: Python 脚本计算（财务指标 → 估值 → 行业对比 + 技术分析并行）
  → Step 3: sentiment-fetcher 采集舆情数据（含小作文URL抓取）
  → Step 3b: 准备小作文/用户想法结构化数据
  → Step 4: 5 个 Agent 并行分析（每个Agent交叉验证小作文/用户想法）
      - Agent A: growth-logic-analyzer（成长逻辑提取）
      - Agent B: leading-indicator-scorer（前置指标评分）
      - Agent C: valuation-industry-analyzer（估值行业分析）
      - Agent D: risk-falsification-analyzer（风险证伪分析）
      - Agent F: technical-analysis（技术分析，基于 Pring 第5版）
  → Step 5: growth-decision-synthesizer 初步综合 → draft report
  → Step 5b: 🔔 第二次询问 — 展示初步结论，问是否有新小作文/新想法
      - 有 → 抓取新内容 → 重新 synthesis → 更新报告
      - 没有 → 直接出最终报告
  → Step 6: 输出 Markdown 最终报告到 data/reports/
```

## Python 脚本

运行前确保 `ZHITU_TOKEN` 环境变量或 `config/settings.json` 中的 `api.token` 已配置。

```bash
cd scripts

# 基本面分析
python3 financial_metrics_calculator.py --code {CODE} --output ../data/latest/{CODE}_financial.json
python3 valuation_analyzer.py --code {CODE} --output ../data/latest/{CODE}_valuation.json
python3 industry_comparator.py --code {CODE} --output ../data/latest/{CODE}_industry.json

# 技术分析（可并行运行）
python3 run_technical_analysis.py {CODE} --json ../data/latest/{CODE}_technical.json --no-save
```

## 免责声明

所有输出不构成投资建议。成长股投资胜率可能低于 30-50%，技术分析信号同样存在假阳性，投资者应独立判断。

---
> Source: [WorkingMoekami/growth-stock-system](https://github.com/WorkingMoekami/growth-stock-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
