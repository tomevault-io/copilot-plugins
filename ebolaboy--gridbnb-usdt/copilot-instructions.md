## gridbnb-usdt

> > **最后更新**: 2025-10-17 14:50:00

# GridBNB-USDT 项目 AI 上下文文档

> **最后更新**: 2025-10-17 14:50:00
> **状态**: 生产环境运行中
> **版本**: 主分支

## 变更记录 (Changelog)

| 日期 | 变更内容 | 影响范围 |
|------|---------|---------|
| 2025-10-17 14:50 | 添加 Web 监控界面详解和 API 使用指南 | CLAUDE.md |
| 2025-10-17 14:45 | 完整扫描 monitor.py 和 web_server.py，更新文档 | monitor.py, web_server.py, CLAUDE.md, index.json |
| 2025-10-17 14:36 | 初始化 AI 上下文文档 | 全局 |

---

## 项目愿景

GridBNB-USDT 是一个基于 Python 的**企业级自动化交易系统**，专为币安 (Binance) 交易所设计。采用先进的网格交易策略，结合动态波动率分析和多层风险管理，旨在稳定捕捉市场波动收益。

**核心价值主张**：
- 🚀 支持任意多币种并发交易（BNB/USDT, ETH/USDT, BTC/USDT 等）
- 🧠 智能网格策略：基于52日年化波动率和 EWMA 混合算法
- 🛡️ 多层风险管理：仓位限制、连续失败保护、实时监控
- 🌐 企业级部署：Docker 容器化、Nginx 反向代理、健康检查
- 📱 现代化 Web 界面：实时监控、多币种视图、响应式设计

---

## 架构总览

### 系统层次结构

```
GridBNB-USDT/
├── 核心交易层 (Core Trading Layer)
│   ├── main.py                 # 应用入口，多币种并发管理
│   ├── trader.py               # 网格交易核心逻辑（2042行）
│   └── exchange_client.py      # 币安 API 封装（542行）
├── 策略层 (Strategy Layer)
│   ├── position_controller_s1.py  # S1辅助策略（52日高低点调仓）
│   └── risk_manager.py         # 高级风险管理器
├── 支持层 (Support Layer)
│   ├── order_tracker.py        # 订单跟踪与历史管理
│   ├── monitor.py              # 交易监控
│   └── helpers.py              # 工具函数与通知
├── 配置层 (Configuration Layer)
│   ├── config.py               # 统一配置管理（Pydantic）
│   └── .env                    # 环境变量配置（敏感信息）
├── 接口层 (Interface Layer)
│   └── web_server.py           # Web 监控界面（aiohttp）
├── 部署层 (Deployment Layer)
│   ├── docker-compose.yml      # 容器编排
│   ├── Dockerfile              # 容器镜像定义
│   └── nginx/nginx.conf        # 反向代理配置
└── 测试层 (Testing Layer)
    └── tests/                  # 单元测试与集成测试
```

### 模块结构图

```mermaid
graph TD
    A["(根) GridBNB-USDT"] --> B["核心交易层"];
    A --> C["策略层"];
    A --> D["支持层"];
    A --> E["配置层"];
    A --> F["接口层"];
    A --> G["部署层"];
    A --> H["测试层"];

    B --> B1["main.py"];
    B --> B2["trader.py"];
    B --> B3["exchange_client.py"];

    C --> C1["position_controller_s1.py"];
    C --> C2["risk_manager.py"];

    D --> D1["order_tracker.py"];
    D --> D2["monitor.py"];
    D --> D3["helpers.py"];

    E --> E1["config.py"];
    E --> E2[".env"];

    F --> F1["web_server.py"];

    G --> G1["docker-compose.yml"];
    G --> G2["Dockerfile"];
    G --> G3["nginx/"];

    H --> H1["tests/"];

    click B2 "#trader-模块" "查看 trader 模块详情"
    click B3 "#exchange-client-模块" "查看 exchange_client 模块详情"
    click C1 "#position-controller-s1-模块" "查看 S1 策略模块详情"
```

---

## 模块索引

| 模块名称 | 路径 | 职责 | 关键类/函数 | 行数 |
|---------|------|------|-----------|------|
| **主程序** | `main.py` | 应用入口，多币种并发管理 | `main()`, `run_trader_for_symbol()`, `periodic_global_status_logger()` | 157 |
| **网格交易器** | `trader.py` | 网格交易核心逻辑 | `GridTrader` | 2042 |
| **交易所客户端** | `exchange_client.py` | 币安 API 封装与时间同步 | `ExchangeClient` | 542 |
| **S1仓位控制** | `position_controller_s1.py` | 基于52日高低点的辅助策略 | `PositionControllerS1` | 319 |
| **风险管理器** | `risk_manager.py` | 仓位限制与风控状态管理 | `AdvancedRiskManager`, `RiskState` | 142 |
| **订单跟踪器** | `order_tracker.py` | 订单记录与交易历史管理 | `OrderTracker`, `OrderThrottler` | 314 |
| **Web服务器** | `web_server.py` | 实时监控界面与 API 端点 | `start_web_server()`, `handle_status()`, `handle_log()`, `IPLogger` | 698 |
| **配置管理** | `config.py` | 统一配置与验证 | `Settings`, `TradingConfig` | 208 |
| **辅助函数** | `helpers.py` | 日志、通知、格式化 | `send_pushplus_message()`, `LogConfig` | 151 |
| **监控器** | `monitor.py` | 交易监控逻辑与状态采集 | `TradingMonitor` | 100 |

---

## 运行与开发

### 快速启动

#### Docker 部署（推荐）
```bash
# 1. 克隆项目
git clone https://github.com/EBOLABOY/GridBNB-USDT.git
cd GridBNB-USDT

# 2. 配置环境变量
cp .env.example .env
# 编辑 .env 文件，填入 API 密钥

# 3. 启动服务（Windows）
start-with-nginx.bat

# 启动服务（Linux/Mac）
chmod +x start-with-nginx.sh
./start-with-nginx.sh

# 4. 访问 Web 界面
# http://localhost
```

#### Python 直接运行
```bash
# 1. 创建虚拟环境
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .\.venv\Scripts\activate  # Windows

# 2. 安装依赖
pip install -r requirements.txt

# 3. 配置并运行
cp .env.example .env
# 编辑 .env 文件
python main.py
```

### 环境要求
- **Python**: 3.8+ (推荐 3.10+)
- **Docker**: 20.10+ (可选，推荐生产环境)
- **内存**: 最低 512MB，推荐 1GB+
- **网络**: 稳定互联网连接，建议低延迟到币安服务器

### 核心依赖
```
ccxt>=4.1.0           # 统一交易所 API
numpy>=1.26.0         # 数值计算
pandas>=2.2.0         # 数据分析
aiohttp>=3.9.1        # 异步 HTTP 客户端
python-dotenv>=1.0.0  # 环境变量管理
pydantic>=2.5.0       # 数据验证
loguru>=0.7.2         # 日志管理
```

### 配置说明

**必填配置** (`.env`)：
```bash
# 币安 API
BINANCE_API_KEY="your_api_key"
BINANCE_API_SECRET="your_api_secret"

# 交易对列表（逗号分隔）
SYMBOLS="BNB/USDT,ETH/USDT,BTC/USDT"

# 交易对特定参数（JSON 格式）
INITIAL_PARAMS_JSON='{"BNB/USDT": {"initial_base_price": 683.0, "initial_grid": 2.0}}'

# 最小交易金额
MIN_TRADE_AMOUNT=20.0
```

**可选配置**：
```bash
# 初始本金（用于收益计算）
INITIAL_PRINCIPAL=800

# 理财功能开关（子账户用户建议设为 false）
ENABLE_SAVINGS_FUNCTION=true

# PushPlus 通知
PUSHPLUS_TOKEN="your_pushplus_token"

# Web UI 认证
WEB_USER=admin
WEB_PASSWORD=your_password
```

---

## 测试策略

### 测试文件结构
```
tests/
├── __init__.py
├── test_config.py          # 配置验证测试
├── test_trader.py          # 交易器核心逻辑测试
├── test_risk_manager.py    # 风险管理测试
└── test_web_auth.py        # Web 认证测试
```

### 运行测试
```bash
# 运行所有测试
python run_tests.py

# 或使用 pytest
pytest tests/

# 运行特定测试文件
pytest tests/test_trader.py -v
```

### 测试覆盖的关键场景
- ✅ 配置加载与验证
- ✅ 网格交易信号检测
- ✅ 风险管理状态转换
- ✅ Web 界面认证机制
- ⚠️ **缺失**：交易所 API 模拟测试、S1 策略单元测试

---

## 编码规范

### Python 代码风格
- 遵循 PEP 8 规范
- 使用 4 空格缩进
- 类名使用 PascalCase（如 `GridTrader`）
- 函数名使用 snake_case（如 `execute_order`）
- 私有方法前缀 `_`（如 `_get_latest_price`）

### 异步编程约定
- 所有 I/O 操作使用 `async/await`
- 避免阻塞操作在主事件循环中
- 使用 `asyncio.gather()` 进行并发任务管理

### 日志记录规范
```python
# 使用 logging 模块，级别分层：
# DEBUG: 详细调试信息（波动率计算、缓存命中）
# INFO:  正常运行日志（交易执行、网格调整）
# WARNING: 警告信息（余额不足、重试操作）
# ERROR: 错误信息（API 调用失败、异常捕获）
# CRITICAL: 严重错误（连续失败、系统停止）

self.logger.info(f"交易执行成功 | 价格: {price} | 数量: {amount}")
```

### 错误处理原则
1. **外层捕获**：主循环捕获所有异常，避免程序崩溃
2. **重试机制**：API 调用失败时自动重试（最多 3-10 次）
3. **降级策略**：关键数据获取失败时使用缓存或默认值
4. **通知告警**：严重错误时通过 PushPlus 发送通知

---

## AI 使用指引

### 代码导航快捷路径
- **交易逻辑核心**：`trader.py` → `main_loop()` 方法（第 553-650 行）
- **网格信号检测**：`trader.py` → `_check_buy_signal()`, `_check_sell_signal()` 方法
- **订单执行流程**：`trader.py` → `execute_order()` 方法（第 796-945 行）
- **风控判断**：`risk_manager.py` → `check_position_limits()` 方法
- **API 调用封装**：`exchange_client.py` → 各 `fetch_*` 和 `create_*` 方法

### 常见问题定位

**问题1：订单执行失败**
- 检查路径：`trader.py::execute_order()` → `exchange_client.py::create_order()`
- 日志关键词：`下单失败`, `Insufficient balance`, `时间同步错误`

**问题2：理财功能报错**
- 检查配置：`.env` 中 `ENABLE_SAVINGS_FUNCTION` 是否为 `true`
- 检查路径：`exchange_client.py::transfer_to_savings()`, `transfer_to_spot()`
- 注意：子账户用户需禁用理财功能

**问题3：多币种运行异常**
- 检查路径：`main.py::main()` → `run_trader_for_symbol()`
- 验证：所有交易对的计价货币必须一致（如都是 USDT）
- 日志关键词：`计价货币不一致`

### 修改策略指南

**调整网格参数**：
```python
# 修改文件：config.py
# 位置：TradingConfig 类 → GRID_PARAMS 字典
GRID_PARAMS = {
    'initial': 2.0,  # 初始网格大小 (%)
    'min': 1.0,      # 最小网格 (%)
    'max': 4.0,      # 最大网格 (%)
    'volatility_threshold': { ... }  # 波动率映射
}
```

**修改风控阈值**：
```python
# 修改文件：config.py
# 位置：Settings 类固定配置部分
MAX_POSITION_RATIO: float = 0.9  # 最大仓位比例 (90%)
MIN_POSITION_RATIO: float = 0.1  # 最小底仓比例 (10%)
```

**添加新交易对**：
```bash
# 修改文件：.env
# 1. 在 SYMBOLS 中添加
SYMBOLS="BNB/USDT,ETH/USDT,BTC/USDT,SOL/USDT"

# 2. 在 INITIAL_PARAMS_JSON 中配置初始参数（可选）
INITIAL_PARAMS_JSON='{"SOL/USDT": {"initial_base_price": 100.0, "initial_grid": 2.5}}'
```

### 关键数据流

**1. 交易信号生成流程**
```
获取当前价格 → 计算波动率 → 调整网格大小 → 检测买卖信号
→ 风控检查 → 执行订单 → 更新状态 → 资金转移（理财）
```

**2. 仓位控制逻辑**
```
获取账户余额（现货+理财） → 计算仓位比例 → 判断风控状态
→ RiskState.ALLOW_ALL / ALLOW_SELL_ONLY / ALLOW_BUY_ONLY
```

**3. S1 辅助策略触发**
```
每日更新52日高低价 → 检测价格突破 → 判断仓位比例
→ 计算调仓金额 → 执行市价单 → 不更新网格基准价
```

**4. Web 监控数据流**
```
用户访问 Web 页面 → Basic 认证 → 加载 HTML/JS
→ JavaScript 发起 /api/symbols 获取交易对列表
→ 用户选择交易对（或默认第一个）
→ 定时轮询 /api/status?symbol=XXX (每5秒)
→ 更新前端显示（价格、余额、网格参数、交易历史等）
```

---

## Web 监控界面详解

### 访问方式

**本地访问**：
```bash
# Docker 部署（通过 Nginx）
http://localhost

# Python 直接运行
http://localhost:58181
```

**认证配置**：
- 在 `.env` 中设置 `WEB_USER` 和 `WEB_PASSWORD`
- 如果未设置，则无需认证（开发模式）
- 使用 HTTP Basic 认证（浏览器会弹出登录框）

### 界面功能

**1. 多币种切换**
- 页面顶部下拉菜单可切换不同交易对
- 自动加载对应交易对的实时数据
- 页面标题动态更新为当前交易对

**2. 基本信息卡片**
- 交易对名称
- 基准价格（网格中心价）
- 当前市场价格
- S1 策略 52日最高价/最低价
- 当前仓位比例

**3. 网格参数卡片**
- 当前网格大小（百分比）
- 网格上轨价格（USDT）
- 网格下轨价格（USDT）
- 触发阈值
- 目标委托金额

**4. 资金状况卡片**
- 总资产（现货 + 理财）
- 计价货币余额（如 USDT）
- 基础货币余额（如 BNB）
- 总盈亏（USDT）
- 盈亏率（%，绿色为盈利，红色为亏损）

**5. 系统资源监控**
- CPU 使用率
- 内存使用量/总量
- 系统运行时间

**6. 最近交易记录**
- 最近10笔交易
- 显示时间、方向（买/卖）、价格、数量、金额
- 买入显示绿色，卖出显示红色

**7. IP 访问记录**
- 最近5条访问记录
- 显示时间、IP 地址、访问路径
- 相同 IP 只记录最新访问时间

**8. 系统日志**
- 实时倒序显示日志内容
- 深色背景，便于查看

### API 端点说明

#### 1. GET `/` 或 `/{HOME_PREFIX}`
**功能**：返回完整的 Web 监控页面（HTML）

**认证**：需要（如果配置了 WEB_USER 和 WEB_PASSWORD）

**返回**：HTML 页面（包含 TailwindCSS 样式和 JavaScript）

---

#### 2. GET `/api/status?symbol={SYMBOL}`
**功能**：获取指定交易对的实时状态数据

**认证**：需要

**参数**：
- `symbol`（可选）：交易对名称，如 `BNB/USDT`。如果省略，返回第一个交易对的数据。

**返回示例**：
```json
{
  "symbol": "BNB/USDT",
  "base_asset": "BNB",
  "quote_asset": "USDT",
  "base_price": 683.0,
  "current_price": 685.5,
  "grid_size": 0.02,
  "threshold": 0.004,
  "total_assets": 850.25,
  "quote_balance": 120.50,
  "base_balance": 1.0645,
  "target_order_amount": 85.02,
  "trade_history": [
    {
      "timestamp": "2025-10-17 14:30:15",
      "side": "buy",
      "price": 682.5,
      "amount": 0.1234,
      "profit": 0.52
    }
  ],
  "last_trade_price": 682.5,
  "last_trade_time": 1697521815,
  "last_trade_time_str": "2025-10-17 14:30:15",
  "total_profit": 50.25,
  "profit_rate": 6.28,
  "s1_daily_high": 690.0,
  "s1_daily_low": 675.0,
  "position_percentage": 65.5,
  "grid_upper_band": 696.86,
  "grid_lower_band": 669.14,
  "uptime": "2天 5小时 30分钟 15秒",
  "uptime_seconds": 192615
}
```

**字段说明**：
- `total_assets`：全账户总资产（用于盈亏计算）
- `target_order_amount`：单次委托目标金额（交易对资产的10%）
- `position_percentage`：当前仓位比例（基础货币占总资产的百分比）
- `grid_upper_band` / `grid_lower_band`：网格买卖触发价格

---

#### 3. GET `/api/symbols`
**功能**：获取所有正在运行的交易对列表

**认证**：需要

**返回示例**：
```json
{
  "symbols": ["BNB/USDT", "ETH/USDT", "BTC/USDT"]
}
```

---

#### 4. GET `/api/logs`
**功能**：获取系统日志内容（倒序）

**认证**：需要

**返回**：纯文本日志（最新的在前）

---

### 使用 curl 调用 API 示例

```bash
# 1. 获取交易对列表
curl -u admin:password http://localhost:58181/api/symbols

# 2. 获取 BNB/USDT 状态
curl -u admin:password "http://localhost:58181/api/status?symbol=BNB/USDT"

# 3. 获取系统日志（前20行）
curl -u admin:password http://localhost:58181/api/logs | head -20
```

### 监控模块内部实现

**TradingMonitor 类** (`monitor.py:100`)

**核心方法**：

1. `get_current_status()` - 采集交易器状态
   - 安全调用 trader 的私有方法（使用 `hasattr` 检查）
   - 处理异常，避免监控逻辑影响交易主流程
   - 返回包含所有关键指标的字典

2. `add_trade(trade)` - 添加交易记录
   - 验证交易数据结构（必须包含 timestamp, side, price, amount, order_id）
   - 自动限制历史记录大小（最多50条）
   - 使用 FIFO 策略（先进先出）

3. `get_trade_history(limit=10)` - 获取历史记录
   - 返回最近 N 笔交易（默认10笔）

**设计特点**：
- **松耦合**：通过依赖注入接收 trader 实例，便于测试
- **防御式编程**：大量使用 try-except 和 hasattr，确保不会因属性缺失崩溃
- **资源控制**：自动限制历史记录大小，防止内存泄漏

---

## 技术债务与改进方向

### 当前已知问题
1. **测试覆盖不足**：缺少交易所 API 模拟测试和 S1 策略单元测试
2. **日志过于频繁**：部分缓存命中日志使用 INFO 级别，应降为 DEBUG
3. **硬编码参数**：部分魔术数字分散在代码中（如最小申购额 1.0 USDT）

### 优化建议
1. **性能优化**：引入 Redis 缓存替代内存缓存，减少 API 调用频率
2. **可观测性**：集成 Prometheus + Grafana 进行指标监控
3. **策略扩展**：支持自定义策略插件机制
4. **安全加固**：API 密钥使用加密存储，避免明文 `.env`

---

## 相关文件清单

### 核心文件（必读）
- `main.py`：应用入口
- `trader.py`：网格交易核心
- `exchange_client.py`：交易所 API 封装
- `config.py`：配置管理
- `.env.example`：配置模板

### 部署文件
- `docker-compose.yml`：容器编排
- `Dockerfile`：镜像定义
- `nginx/nginx.conf`：反向代理配置
- `start-with-nginx.sh`：启动脚本

### 文档文件
- `README.md`：项目主文档
- `README-https.md`：HTTPS 配置教程
- `CLAUDE.md`：本文件（AI 上下文）

### 数据文件（运行时生成）
- `data/trader_state_*.json`：交易器状态持久化
- `data/trade_history.json`：交易历史记录
- `trading_system.log`：系统日志

---

## 附录：术语表

| 术语 | 定义 |
|------|------|
| **网格交易** | 在价格区间内设置多个买卖点位，自动高抛低吸的策略 |
| **基准价** | 网格策略的中心价格，买卖上下轨以此为基础计算 |
| **波动率** | 价格变动的剧烈程度，用于动态调整网格大小 |
| **EWMA** | 指数加权移动平均，赋予近期数据更高权重的波动率算法 |
| **S1 策略** | 基于52日最高/最低价的辅助仓位控制策略 |
| **风控状态** | 系统根据仓位比例决定的操作限制（允许全部/仅买/仅卖） |
| **理财账户** | 币安活期理财，闲置资金自动申购赚取利息 |
| **现货账户** | 币安现货账户，用于交易的资金池 |

---

**文档生成器**: Claude AI
**联系方式**: [Telegram 群组](https://t.me/+b9fKO9kEOkg2ZjI1) | [GitHub Issues](https://github.com/EBOLABOY/GridBNB-USDT/issues)

---
> Source: [EBOLABOY/GridBNB-USDT](https://github.com/EBOLABOY/GridBNB-USDT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
