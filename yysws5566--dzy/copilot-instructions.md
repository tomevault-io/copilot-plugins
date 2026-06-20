## dzy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git 仓库

- **Remote:** `https://github.com/yysws5566/dzy.git`
- **分支:** `main`
- **每次代码修改完成后，执行以下命令同步：**
  ```bash
  cd "C:\Users\西西家的咩咩\Projects"
  git add -A
  git commit -m "<改动说明>"
  git push
  ```
- **禁止提交的内容（.gitignore 已配置）：** API Key、回测结果 JSON/MD、数据库文件、日志 txt、CSV/XLSX

## 项目概览

西西的 A 股量化交易工作区，共 5 个策略系统 + 1 个增强因子库，彼此有因子复用和复盘关联。

**v9 升级：所有策略已嵌入增强因子库（RSI + KDJ + 威廉%R + 主力资金流 + VWAP + 大单活跃 + 板块强度 + 尾盘共振）**

```
Projects/
├── enhanced_factors.py                ← 🆕 增强因子库（8因子，TickFlow Pro）
├── quant/quant-trading-system/       ← 策略① 多因子扫描（12→15因子 + 回测 + 优化）
│   └── factors/
│       ├── factor1~12_*.py           ← 原有因子
│       ├── factor13_rsi_oversold.py  ← 🆕 RSI超卖反弹
│       ├── factor14_kdj.py           ← 🆕 KDJ低位金叉
│       └── factor15_money_flow.py    ← 🆕 主力资金流向
├── strategies/
│   ├── enhanced_factors.py            ← 🆕 增强因子引擎
│   ├── box-trading/                  ← 策略② 箱体波动（紫金矿业 + 洛阳钼业）
│   ├── overnight-holding/            ← 策略③ 一夜持股法 v9 🆕（12维评分 + 隔天概率）
│   ├── short-term/                   ← 策略④ 短线选股 v8 Pro 🆕（TickFlow + 双引擎）
│   └── morning-screener/             ← 策略⑤ 早盘选股 v2 Pro 🆕（含卖点/止损分析）
├── scripts/                          ← 零散工具脚本
└── sandbox/                          ← 试验性项目
```

**策略演进关系：**
- ③④合并了①②的高胜率因子，升级为更高胜率的尾盘买入策略
- ⑤复盘①②③的误差，自我优化升级，并在开盘 9:00-10:00 用高胜率因子分析卖点和止损点

## 数据源

- **A 股数据全部走 TickFlow Pro**（非免费套餐），Python SDK 已安装
- 用法：`from tickflow import TickFlow` → `client = TickFlow()`（自动读取环境变量）
- 标的格式：`601899.SH`（上海）、`000001.SZ`（深圳）、`8xxxxx.BJ`（北交所）
- 实时行情：`client.quotes.get()`，K线：`client.klines.get()`，分时：`client.klines.intraday()`
- 实时推送：`QuoteStream`（`from tickflow.resources.realtime import QuoteStream`）
- 部分早期脚本使用新浪/AKShare 作为免费备用数据源

---

## 策略① 多因子量化扫描系统 (`quant/quant-trading-system/`)

### 运行命令

```bash
cd quant/quant-trading-system
pip install -r requirements.txt   # 核心仅需 requests，pandas/numpy 可选

python main.py                    # 当日扫描
python main.py --backtest         # 回测模式
python main.py --date 2026-06-05  # 指定日期扫描
```

### 架构

```
交易日判断 → 全市场股票获取 → 流动性初筛 → 12因子计算 → 加权打分 → 信号输出 + 复盘
```

| 模块 | 文件 | 职责 |
|------|------|------|
| 数据层 | `tickflow_client.py`, `data_fetcher.py` | TickFlow SDK 封装 + 降级方案 |
| 筛选层 | `liquidity_filter.py`, `trading_calendar.py` | 流动性初筛 + A股交易日历 |
| 因子层 | `factors/factor1~12_*.py` | 12 个 T+1 短线因子 |
| 扫描层 | `tail_scanner.py`, `merged_scanner.py` | 尾盘 2:40 模式 + 多因子合并 |
| 评分层 | `scorer.py` | 加权打分 → 信号 + 仓位建议 |
| 优化层 | `optimizer.py`, `overnight_optimizer.py` | 权重优化 + 隔夜持仓优化 |
| 回测层 | `backtest.py` | 回测引擎 |
| 复盘层 | `review_engine.py` | 自动复盘，统计胜率和因子表现 |
| 输出层 | `reporter.py` | Markdown 报告 |
| 调度层 | `scheduler.py` | 定时任务 |

**12 因子：** 尾盘量价背离 / 封板质量 / 缺口博弈 / 北向资金背离 / 集合竞价 / 板块反转 / 龙虎榜 / 板块滞后 / 整数关口心理 / 融资情绪 / 大宗交易 / 全球联动

---

## 策略② 箱体波动策略 (`strategies/box-trading/`)

```bash
cd strategies/box-trading
python 箱体监控_实时提醒.py              # 实时监控（TickFlow QuoteStream 推送）
python 箱体监控_实时提醒.py --once       # 快照一次
python 箱体监控_实时提醒.py --backtest   # 回测
python 箱体策略_生成PPT.py               # 生成 PPT 报告
```

**标的：** 紫金矿业 `601899.SH` + 洛阳钼业 `603993.SH`  
**逻辑：** 近 N 日 K 线高低点 + ATR 定箱体边界 → 价格抵近触发买卖 → 分时/日线/周线三级共振 → 每日自动重算

---

## 策略③ 一夜持股法 (`strategies/overnight-holding/`)

```bash
cd strategies/overnight-holding
python overnight_strategy_v9.py          # 🆕 v9.0 增强因子版（TickFlow Pro）
python overnight_strategy_v9.py --debug  # 调试模式
python overnight_strategy_v8.py          # v8.0 旧版（仍可用）
```

**v9.0 核心升级（vs v8）：**
- 涨幅甜点 2.0-4.0%（收窄，避免 >5% 反转风险）
- close/high ≥ 0.97（尾盘强势度）
- 量比 1.2-3.0（识别真实资金介入）
- 盘中走势质量评分（处罚尾盘急拉诱多）
- 八维加权评分（满分 100，<60 不选）
- 大盘多维评估

历史版本在 `versions/` 目录，TOP10 报告在 `reports/` 目录。

---

## 策略④ 短线选股 (`strategies/short-term/`)

```bash
cd strategies/short-term
python short_term_screener_v7.py   # v7.0 主程序（新浪数据源）
python short_term_screener_v5.py   # v5.0（功能最完整，20KB）
```

合并了策略①②的因子，升级为更高胜率的尾盘买入策略。  
`data_fetcher.py`（30KB）是核心数据获取模块。  
历史版本在 `versions/` 目录，每日信号在 `reports/` 目录。

---

## 策略⑤ 早盘选股 (`strategies/morning-screener/`)

```bash
cd strategies/morning-screener
python morning_screener.py          # 早盘主程序（AKShare 数据源）
python morning_screener_westock.py  # Westock 数据源版本
```

**执行时间：** 每日 09:25（集合竞价结束后）  
**核心功能：**
- 6 层过滤筛选早盘标的
- 复盘策略①②③的历史误差，自我优化
- 9:00-10:00 用高胜率因子分析卖点和止损点
- 比尾盘版更激进的筛选参数（允许平开/小涨，涨幅放宽至 8%）

---

## 编码注意事项

- **Windows GBK 环境**：bash 对中文文件名支持有问题。移动/重命名中文文件请用 Python `shutil`/`os.rename`，避免直接用 `mv`。
- 所有策略 `.py` 文件首行有 UTF-8 编码处理（`encoding_fix` 或手动 wrapper）。
- 各策略的 `versions/` 目录保留了历史迭代版本，不要随意删除。

---
> Source: [yysws5566/dzy](https://github.com/yysws5566/dzy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
