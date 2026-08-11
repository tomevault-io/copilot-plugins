## auto-quant

> 本仓库是 **策略研究流水线**：用户给出思想 → 实现并记录 → 回测验证 → 生成报告 → 人工在 Cursor 中调用 AI 分析。

# auto_quant — Cursor Agent 指南

本仓库是 **策略研究流水线**：用户给出思想 → 实现并记录 → 回测验证 → 生成报告 → 人工在 Cursor 中调用 AI 分析。

## 目录约定

| 路径 | 用途 |
|------|------|
| `research/strategies/<ID>/` | 每个策略一个项目文件夹 |
| `research/strategies/<ID>/idea.md` | 用户策略思想（优先阅读） |
| `research/strategies/<ID>/meta.yaml` | 状态、平台、回测区间 |
| `research/strategies/<ID>/strategy.py` | 聚宽实现代码 |
| `research/strategies/<ID>/runs/*.json` | 历次回测原始结果 |
| `research/strategies/<ID>/report.md` | 人类可读研究报告 |
| `research/strategies/<ID>/analysis_request.md` | **给 AI 的分析请求**（用户 @ 此文件） |
| `research/strategies/<ID>/ablation.yaml` | 消融实验：各优化点对应独立 `.py` 与开关 |
| `research/strategies/<ID>/variants/*.py` | 从 `strategy.py` 复制后**只改单一优化点** |
| `research/strategies/<ID>/ablation_report.md` | 各变体指标对比 + 启发式「是否合并」建议 |
| `output/jq_strategies/` | 聚宽策略池（可 import 进 research） |
| `src/auto_quant/strategies/` | 本地可回测策略（如 MACD） |

## 推荐工作流（用户）

```bash
# 1. 新建或导入
python scripts/research.py new "策略名" --idea "买小市值+MA20过滤..."
python scripts/research.py import output/jq_strategies/xxx.py --idea "..."

# 2. 回测
python scripts/research.py run <ID>

# 3. 生成 AI 分析稿
python scripts/research.py report <ID>

# 4. 多优化点消融（与 idea.md 章节对齐）
python scripts/research.py ablation-init <ID>        # 生成 ablation.yaml + variants/README
# 按 README 复制 strategy.py 为 variants/idea-N.py，逐项改代码后 enabled: true
python scripts/research.py ablation-run <ID>        # 依次聚宽回测并写入 runs/*_jq_ablation*.json
python scripts/research.py ablation-report <ID>     # ablation_report.md 对比 + 合并建议
```

消融分析时额外阅读 **`ablation_report.md`** 与 `runs/*_jq_ablation*.json`，对照 `idea.md` 各节「预期效果」判断合并是否合理。

用户在 Cursor 聊天中说：**请阅读 `@research/strategies/<ID>/analysis_request.md` 并给出评审**。

## Agent 分析时请做到

1. 阅读 `idea.md`、`meta.yaml`、最新 `runs/*.json`、`report.md`
2. 若平台为 `joinquant`，阅读 `strategy.py`，并对照 **joinquant-skill** 检查 API 用法、未来函数、交易成本
3. 对 `local_macd` 平台，阅读 `src/auto_quant/strategies/` 相关实现
4. 输出：逻辑有效性、回测可信度、风险、可执行性、改进建议、是否进入下一轮
5. 若存在消融实验：阅读 `ablation_report.md` 与各 `runs/*_jq_ablation*.json`，评估「合并建议」是否成立（启发式规则不能替代人工）
6. 若用户要求改代码，修改 `strategy.py` 或本地策略后，提醒运行 `python scripts/research.py run <ID>`

## 状态机

`draft` → `implemented`（已挂载代码）→ `backtested`（已跑回测）→ `reviewed`（AI/人工评审通过）→ `archived`

## 禁止

- 不要编造回测数字；以 `runs/*.json` 和聚宽网页为准
- 不要跳过免责声明：回测不代表未来收益

---
> Source: [xzcawl/auto_quant](https://github.com/xzcawl/auto_quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
