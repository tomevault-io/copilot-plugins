## ma30-dca-strategy

> >


# 30均线趋势过滤器 × 定投策略 Skill

## 概述

本 Skill 实现了 **MA30 趋势过滤器联合定投** 的完整回测工作流，包含数据获取、四种策略回测、交互式 HTML 报告生成三个阶段。

---

## 使用场景

当用户提到以下需求时，自动加载本 Skill：

- "帮我做定投回测"
- "30均线定投策略"
- "MA30趋势过滤定投"
- "分析定投策略表现"
- "智能定投 / 动态定投"
- 指定标的（如"用沪深300ETF做定投回测"）

---

## 工作流程

### Phase 1 — 数据获取

调用 `finance-data-retrieval` skill 获取目标 ETF/基金/股票的历史日线数据。

**默认标的**：沪深300ETF（510300.SH）  
**默认区间**：近5年  
**API**：`fund_daily`（ETF）或 `daily`（股票）

```
参数示例：
  ts_code: "510300.SH"
  start_date: "20200101"
  end_date: "今天"
  fields: "ts_code,trade_date,open,high,low,close,vol,amount"
```

将数据保存为 JSON 文件，供回测脚本读取。

---

### Phase 2 — 策略回测

执行 `scripts/backtest.py` 进行四种策略的完整回测。

**四种策略说明**：

| 策略 | 逻辑 |
|------|------|
| 普通定投 | 每月第一交易日固定投入基础金额，不做任何调整 |
| 趋势加倍定投 | 收盘 < MA30 → 2倍投入；收盘 ≥ MA30 → 1倍投入 |
| 趋势暂停定投 | 收盘 < MA30 → 暂停投入；收盘 ≥ MA30 → 正常投入 |
| 趋势分级定投 | 按价格偏离MA30程度动态调整倍数（0.5x ~ 3x） |

**分级定投档位**：
- 均线上方 ≥5%：× 0.5（减半）
- 均线上方 0-5%：× 1.0（正常）
- 均线下方 0-5%：× 1.5
- 均线下方 5-10%：× 2.0
- 均线下方 >10%：× 3.0

**运行命令**：
```bash
python scripts/backtest.py \
  --data_path <数据JSON路径> \
  --output_dir <输出目录> \
  --monthly_amount 1000 \
  --ma_period 30
```

---

### Phase 3 — HTML 报告生成

执行 `scripts/generate_report.py` 生成交互式 HTML 回测报告。

**报告内容**：
1. 四策略对比卡片（可点击切换）
2. 当前策略核心指标（总投入、最终市值、绝对盈亏、年化收益、最大回撤）
3. 价格走势 + MA30 均线图（含缩放）
4. 组合市值 vs 累计投入曲线
5. 四策略收益率动态对比折线图
6. 每月实际投入柱状图（颜色区分加倍/正常/暂停）
7. 策略综合对比表格
8. 交易明细记录（可按买入/暂停过滤）

**运行命令**：
```bash
python scripts/generate_report.py \
  --results_path <回测结果JSON路径> \
  --price_path <价格MA30JSON路径> \
  --output_path <HTML输出路径>
```

---

## 完整运行示例

```
用户: 帮我对沪深300ETF做30均线定投策略回测，基础金额1000元

AI 执行流程:
1. [finance-data-retrieval] 获取 510300.SH 历史日线数据
2. [backtest.py] 运行四种策略回测，输出 backtest_results.json + price_ma30.json
3. [generate_report.py] 生成 ma30_dca_report.html
4. [preview_url] 在浏览器中预览报告
5. 输出回测摘要表格
```

---

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `ts_code` | 标的代码 | `510300.SH` |
| `start_date` | 回测开始日期 | 5年前 |
| `end_date` | 回测结束日期 | 今天 |
| `monthly_amount` | 每期基础定投金额（元） | `1000` |
| `ma_period` | 均线周期（交易日） | `30` |
| `output_dir` | 输出目录 | 工作区根目录 |

---

## 输出文件

| 文件 | 说明 |
|------|------|
| `<标的>_data.json` | 原始行情数据 |
| `backtest_results.json` | 四策略回测结果（含净值序列、交易日志） |
| `price_ma30.json` | 价格 + MA30 序列（供图表使用） |
| `ma30_dca_report.html` | 交互式回测报告（直接用浏览器打开） |

---

## 注意事项

1. **数据获取**：ETF 用 `fund_daily` API，股票用 `daily` API，确认标的类型后选择正确接口
2. **MA30 有效性**：前29个交易日无法计算均线，这些日期的定投统一按普通定投处理
3. **分红复权**：当前使用不复权数据，如需复权数据请在数据获取时指定 `adj='qfq'`（前复权）
4. **本地服务器**：预览 HTML 时需启动本地 HTTP 服务，端口默认 8899

---

## 依赖

- Python 3.8+（无需额外安装第三方库，仅使用标准库 `json`, `math`, `datetime`）
- ECharts 5.4.3（CDN 加载，需要网络）
- `finance-data-retrieval` Skill（数据获取阶段）

---
> Source: [Leck88/ma30-dca-strategy](https://github.com/Leck88/ma30-dca-strategy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
