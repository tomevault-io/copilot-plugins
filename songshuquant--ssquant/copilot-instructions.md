## ssquant

> SSQuant 是一个专业的 **中国期货 CTP 量化交易框架**（**v0.4.4**），基于 Python 构建，涵盖回测、仿真（SIMNOW）和实盘全链路。

# SSQuant — AI Agent 项目上下文

## 项目概述

SSQuant 是一个专业的 **中国期货 CTP 量化交易框架**（**v0.4.4**），基于 Python 构建，涵盖回测、仿真（SIMNOW）和实盘全链路。

**v0.4.4 要点**：回测侧交易配对/权益/滑点/盈亏比等统计修复；多数据源支持 `capital_ratio` 比例分配资金；data_server 历史 K 线 HTTP 与 `fallback_servers` 与鉴权一致轮询；详见仓库根目录 **`更新日志_v0.4.4.md`**。

## 技术栈

- **语言**: Python 3.9–3.14
- **CTP 接口**: 上期技术 CTP 6.7.7 及以上版本（Windows x64 + Linux x64）
- **数据**: REST API + WebSocket（data_server 模式）、本地复权算法
- **核心依赖**: pandas, numpy, websockets, requests

## 目录结构

```
ssquant/
├── api/              # StrategyAPI — 策略唯一入口
│   └── strategy_api.py
├── backtest/         # 回测引擎、自动移仓、复权审计
│   ├── backtester.py
│   ├── rollover_engine.py
│   └── rollover_audit.py
├── config/           # 交易配置（回测/仿真/实盘）
│   └── trading_config.py
├── ctp/              # CTP 动态库加载
├── data/             # 数据源、本地复权、合约映射
│   ├── data_source.py
│   ├── local_adjust.py
│   └── contract_mapper.py
├── indicators/       # 技术指标
├── pyctp/            # CTP 客户端（SIMNOW / 实盘）
│   ├── simnow_client.py
│   └── real_trading_client.py
└── __init__.py       # __version__ = "0.4.4"
examples/             # 25+ 示例策略（B_回测 / A_工具 / C_高级 / D_数据）
```

## 核心模式

| 运行模式 | RunMode 枚举 | 用途 |
|---------|-------------|------|
| 回测 | `RunMode.BACKTEST` | 历史数据离线回测 |
| SIMNOW | `RunMode.SIMNOW` | 上期 SIMNOW 仿真交易 |
| 实盘 | `RunMode.REAL_TRADING` | 期货公司真实交易 |

## 策略生命周期

```python
from ssquant.api.strategy_api import StrategyAPI
from ssquant.config.trading_config import get_config, RunMode

def initialize(api: StrategyAPI):
    """一次性初始化（可选）"""
    api.log("策略启动")

def strategy(api: StrategyAPI):
    """每根K线收盘 / 每个Tick 调用"""
    klines = api.get_klines()
    if len(klines) < 20:
        return
    price = api.get_price()
    pos = api.get_pos()
    # 交易逻辑 ...
    if should_buy:
        api.buy(volume=1, reason="信号触发")

config = get_config(RunMode.BACKTEST, symbol="rb888", ...)
```

## 关键约定

1. **所有交易操作**通过 `StrategyAPI` 进行：`buy / sell / sellshort / buycover / close_all`
2. **连续合约**: `888`（主力连续）、`777`（次主力）、`000`（指数合约）
3. **复权类型**: `adjust_type='0'`(不复权) / `'1'`(后复权) / `'2'`(前复权)
4. **移仓模式**: `simultaneous`（同时平旧开新）/ `sequential`（先平后开）
5. **data_server 模式**: 服务器推送任意周期 K 线 + 订单流，无需本地聚合；HTTP 鉴权与历史 K 线请求均遵循 `api_url` + `fallback_servers` 顺序（v0.4.4+）
6. **不再支持 PyPI**，仅通过 Git 仓库安装更新

## AI Agent 技能

详见 `.claude/skills/` 目录下的 5 个技能模块：

| 技能 | 目录 | 能力 |
|------|------|------|
| 策略编写 | `ssquant-strategy` | 编写符合框架规范的期货策略 |
| 回测验证 | `ssquant-backtest` | 配置并执行回测、分析结果 |
| 部署上线 | `ssquant-deploy` | SIMNOW 仿真 → 实盘部署 |
| 问题诊断 | `ssquant-debug` | 排查 CTP、数据、策略异常 |
| 数据查询 | `ssquant-data` | 查询行情、持仓、账户数据；含 data_server 与备用节点 |

各技能包 frontmatter 中 `version` 与框架 **v0.4.4** 对齐；发布说明见 **`更新日志_v0.4.4.md`**。

---
> Source: [songshuquant/ssquant](https://github.com/songshuquant/ssquant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
