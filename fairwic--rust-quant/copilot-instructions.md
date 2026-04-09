## rust-quant

> ├── common/          # 通用工具 (3+包使用)


# Rust Quant 代码放置规范

## 📦 包结构快速参考

```
crates/
├── common/          # 通用工具 (3+包使用)
├── core/            # 核心基础设施 (配置、日志、数据库连接池)
├── domain/          # 领域模型 (实体、值对象、枚举、接口)
├── infrastructure/  # 基础设施实现 (DB/Redis/消息/交易所适配/Repository)
├── services/        # 应用服务层 (UseCase：编排业务流程)
├── market/          # 行情接入(Adapter)：REST/WS 拉取、归一化、DTO（不负责持久化）
├── indicators/      # 技术指标计算 (纯函数)
├── strategies/      # 策略算法与框架 (纯业务：不做I/O、不依赖基础设施)
├── risk/            # 风险管理 (仓位、订单、账户风控)
├── execution/       # 订单执行 (订单创建、提交、跟踪)
├── orchestration/   # 任务调度 (定时任务、工作流)
├── analytics/       # 分析报告 (性能分析、报表)
├── ai-analysis/     # AI分析 (情绪分析、事件检测)
└── rust-quant-cli/  # 程序入口 (main.rs)
```

---

## ✅ 推荐目录规范 V3（更少层级/更少重复名词/依赖更硬）

目标：让“放哪里”只看 **职责 + 是否 I/O** 就能决定，不用记一堆 service/repository/model 的重复词。

### 核心思想：Ports / Adapters
- **Port（端口）**：抽象接口，放在稳定的一侧（`domain` 或 `services`）
- **Adapter（适配器）**：具体实现，放在 I/O 一侧（`infrastructure` 或 `market`）

### 强制命名去重（减少认知噪音）
- **services 层不再到处出现 `*_service.rs`**
  - ✅ 推荐：`services/src/usecases/<bounded_context>/<verb>.rs`
  - ✅ 例：`services/src/usecases/strategy/execute.rs`
- **infrastructure 层不再到处出现 `*_repository.rs` 作为唯一形态**
  - ✅ 推荐：`infrastructure/src/storage/<db|redis>/<aggregate>.rs`
  - ✅ 推荐：`infrastructure/src/adapters/<exchange|email|notify>/<name>.rs`
- **market 层只保留“接入”语义**
  - ✅ 推荐：`market/src/ingest/<exchange>/...`、`market/src/streams/...`、`market/src/dto/...`
  - ❌ 禁止出现 `sqlx::FromRow` / `Sqlx*Repository`（持久化全部在 infrastructure）

### 两种落地策略（你可以二选一）
1) **保留 `market` crate（推荐短期）**
   - `market`：只负责拉取/解码/归一化（DTO）
   - `infrastructure`：负责落库/缓存/消息/交易所签名等 I/O
2) **合并 `market` → `infrastructure`（推荐长期）**
   - 把 `market` 作为 `infrastructure/src/adapters/market_ingest/...` 子模块
   - 好处：减少 crate 数、减少“exchanges/repositories”重复名词

---

## 🎯 代码放置决策树

### Q1: 这是业务逻辑还是基础设施？
- **基础设施** → `infrastructure/` 或 `core/`
- **业务逻辑** → 继续 Q2

### Q2: 是纯粹的领域概念吗？
- **是** → `domain/`
- **否** → 继续 Q3

### Q3: 是哪种业务逻辑？
- **技术指标计算** → `indicators/`
- **策略信号生成** → `strategies/`
- **风险控制** → `risk/`
- **订单执行** → `execution/`
- **任务调度** → `orchestration/`
- **分析报告** → `analytics/`
- **AI分析** → `ai-analysis/`
- **市场数据** → `market/`
- **业务协调** → `services/`

### Q4: 是通用工具吗？
- **通用 (3+包使用)** → `common/`
- **配置/基础设施** → `core/`

---

## 📁 详细目录映射

### `domain/` - 领域模型
**职责**: 纯粹的业务逻辑，零外部依赖

**放置规则**:
- ✅ 业务实体 → `domain/src/entities/xxx.rs`
- ✅ 值对象 (Price, Volume) → `domain/src/value_objects/xxx.rs`
- ✅ 业务枚举 → `domain/src/enums/xxx_enums.rs`
- ✅ 领域接口 → `domain/src/traits/xxx_trait.rs`
- ✅ 业务错误 → `domain/src/entities/xxx.rs` 或 `domain/src/value_objects/xxx.rs`

**禁止**:
- ❌ sqlx, redis, tokio 等框架依赖
- ❌ 数据库操作、网络请求、文件IO

---

### `infrastructure/` - 基础设施实现
**职责**: 实现domain接口，数据访问、缓存

**放置规则**:
- ✅ Repository实现 → `infrastructure/src/repositories/xxx_repository.rs`
- ✅ 数据库Entity (FromRow) → `infrastructure/src/repositories/xxx_repository.rs`
- ✅ 缓存实现 → `infrastructure/src/cache/xxx_cache.rs`
- ✅ 消息传递 → `infrastructure/src/messaging/xxx.rs`
- ✅ 交易所适配(REST/WS底层) → `infrastructure/src/exchanges/xxx_adapter.rs`

**要求**:
- ✅ 必须实现 `domain::traits` 中的接口
- ✅ Entity → Domain 转换方法: `to_domain()`
- ✅ Domain → Entity 转换方法: `from_domain()`

---

### `services/` - 应用服务层
**职责**: 协调多个领域对象，实现复杂业务流程

**放置规则**:
- ✅ 用例（推荐）→ `services/src/usecases/<bounded_context>/<verb>.rs`
- ✅ 领域分组（兼容现状）→ `services/src/{strategy,market,risk,trading}/...`

**要求**:
- ✅ **依赖domain接口**: 使用 `domain::traits::XxxRepository` (trait)，而不是infrastructure的具体实现
- ✅ **依赖注入**: 通过构造函数注入infrastructure的具体实现，但字段类型声明为trait类型
- ✅ 协调多个domain聚合根
- ✅ 定义事务边界

**示例**:
```rust
// ✅ 正确: 依赖domain接口
use rust_quant_domain::traits::CandleRepository;
use rust_quant_infrastructure::repositories::SqlxCandleRepository;

pub struct CandleService {
    repository: Box<dyn CandleRepository>,  // trait类型
}

impl CandleService {
    pub fn new(repo: SqlxCandleRepository) -> Self {
        Self { repository: Box::new(repo) }  // 注入具体实现
    }
}

// ❌ 错误: 直接依赖infrastructure具体类型
pub struct CandleService {
    repository: SqlxCandleRepository,  // ❌ 违反依赖倒置
}
```

---

### `strategies/` - 策略引擎
**职责**: 策略框架、具体策略实现、回测

**放置规则**:
- ✅ 策略框架 → `strategies/src/framework/xxx.rs`
- ✅ 具体策略实现 → `strategies/src/implementations/xxx_executor.rs`
- ✅ 回测引擎 → `strategies/src/backtesting/xxx.rs`
- ✅ 策略缓存 → `strategies/src/cache/xxx.rs`

**要求**:
- ✅ 实现 `domain::traits::Strategy` 接口
- ✅ 使用 `indicators` 计算指标
- ✅ 返回 `domain::SignalResult`

**禁止**:
- ❌ 任何数据库/缓存/网络I/O（包括 sqlx/redis/okx/reqwest/tokio-tungstenite）
- ❌ 依赖 `infrastructure/`、`market/`、`services/`、`orchestration/`

---

### `indicators/` - 技术指标
**职责**: 纯计算逻辑，无业务逻辑

**放置规则**:
- ✅ 趋势指标 → `indicators/src/trend/xxx.rs`
- ✅ 动量指标 → `indicators/src/momentum/xxx.rs`
- ✅ 波动率指标 → `indicators/src/volatility/xxx.rs`
- ✅ 成交量指标 → `indicators/src/volume/xxx.rs`
- ✅ 形态识别 → `indicators/src/pattern/xxx.rs`
- ✅ 复杂指标系统 → `indicators/src/trend/xxx/` (目录)

**要求**:
- ✅ 纯函数，无副作用
- ✅ 接受 `&[Candle]` 或 `&[f64]`
- ✅ 返回明确的类型

**禁止**:
- ❌ 数据库操作、缓存操作、策略决策

---

### `risk/` - 风险管理
**职责**: 仓位、订单、账户风控

**放置规则**:
- ✅ 订单风控 → `risk/src/order/xxx.rs`
- ✅ 持仓风控 → `risk/src/position/xxx.rs`
- ✅ 账户风控 → `risk/src/account/xxx.rs`
- ✅ 风控策略 → `risk/src/policies/xxx_policy.rs`
- ✅ 回测模型 → `risk/src/backtest/xxx.rs`

**要求**:
- ✅ 风控规则集中在 `policies/`（纯规则优先：可测、可复用）
- ✅ 若需要持久化/查询：放到 `infrastructure/` 实现对应 repository（risk 不直接写 sqlx）

**禁止**:
- ❌ 策略信号生成 (属于strategies)

---

### `execution/` - 订单执行
**职责**: 订单创建、提交、跟踪、撤单

**放置规则**:
- ✅ 执行引擎 → `execution/src/execution_engine/xxx.rs`
- ✅ 订单管理 → `execution/src/order_manager/xxx_service.rs`
- ✅ 持仓管理 → `execution/src/position_manager/xxx.rs`

**要求**:
- ✅ 只定义“执行流程/状态机/重试策略”等业务规则
- ✅ 调用交易所 API / SDK 属于基础设施：通过 `domain::traits::ExchangeClient`（或类似 trait）注入 `infrastructure` 实现
- ✅ 使用 risk 的订单/仓位/风控输出（但不实现风控规则）

**禁止**:
- ❌ 策略决策 (属于strategies)
- ❌ 风控逻辑 (属于risk)

---

### `market/` - 行情接入（Adapter）
**职责**: REST/WS 数据拉取、归一化、DTO；不负责持久化/Repository

**放置规则**:
- ✅ 网络DTO/归一化模型 → `market/src/models/xxx_dto.rs`
- ✅ WebSocket流（只负责连接与解码，不落库）→ `market/src/streams/websocket_service.rs`
- ✅ 交易所行情适配（到 DTO/Domain 的映射）→ `market/src/exchanges/xxx.rs`

**要求**:
- ✅ 只做“接入与转换”，不做业务编排（编排在 services）
- ✅ 不出现 `sqlx::FromRow` / `Sqlx*Repository`（持久化在 infrastructure）

**禁止**:
- ❌ 直接数据库操作、缓存操作（落库/缓存属于 infrastructure）
- ❌ 策略逻辑、风险控制逻辑、下单逻辑

---

### `orchestration/` - 任务调度
**职责**: 定时任务、工作流编排、事件驱动

**放置规则**:
- ✅ 任务调度 → `orchestration/src/scheduler/xxx_scheduler.rs`
- ✅ 工作流 → `orchestration/src/workflow/xxx_job.rs`
- ✅ 事件总线 → `orchestration/src/event_bus/xxx.rs`
- ✅ 策略运行器 → `orchestration/src/strategy_runner/xxx.rs`

**要求**:
- ✅ 只做编排，不包含业务逻辑
- ✅ 调用services或业务包

**禁止**:
- ❌ 直接实现策略逻辑
- ❌ 直接数据库操作

---

### `core/` - 核心基础设施
**职责**: 配置管理、日志系统、数据库连接池

**放置规则**:
- ✅ 配置管理 → `core/src/config/xxx.rs`
- ✅ 数据库连接池 → `core/src/database/sqlx_pool.rs`
- ✅ Redis客户端 → `core/src/cache/redis_client.rs`
- ✅ 日志系统 → `core/src/logger/xxx.rs`
- ✅ 错误类型 → `core/src/error/mod.rs`

**禁止**:
- ❌ 业务逻辑、具体的业务模型

---

### `common/` - 通用工具
**职责**: 被3+包使用的通用工具

**放置规则**:
- ✅ 通用工具 → `common/src/utils/xxx.rs`
- ✅ 通用类型 → `common/src/types/xxx.rs`
- ✅ 通用枚举 → `common/src/constants/xxx_enums.rs`
- ✅ 通用错误 → `common/src/errors/mod.rs`

**要求**:
- ✅ 必须被3+包使用
- ✅ 无业务逻辑

---

### `analytics/` - 分析报告
**职责**: 性能分析、报表生成

**放置规则**:
- ✅ 性能分析 → `analytics/src/performance/xxx.rs`
- ✅ 报表生成 → `analytics/src/reporting/xxx.rs`

**要求**:
- ✅ 只读分析，不修改业务数据

---

### `ai-analysis/` - AI分析
**职责**: 情绪分析、事件检测、影响预测

**放置规则**:
- ✅ 情绪分析 → `ai-analysis/src/sentiment_analyzer/xxx.rs`
- ✅ 事件检测 → `ai-analysis/src/event_detector/xxx.rs`
- ✅ 影响预测 → `ai-analysis/src/market_impact_predictor/xxx.rs`

---

### `rust-quant-cli/` - 程序入口
**职责**: 应用程序入口、命令行参数解析

**放置规则**:
- ✅ 程序入口 → `rust-quant-cli/src/main.rs`
- ✅ 启动逻辑 → `rust-quant-cli/src/app/bootstrap.rs`
- ✅ CLI库 → `rust-quant-cli/src/lib.rs`

**要求**:
- ✅ 只做入口，参数解析、程序启动
- ✅ 依赖orchestration

**禁止**:
- ❌ 业务逻辑

---

## 🔗 依赖规则

### 依赖方向 (单向，从上到下)
```
cli → orchestration → services → domain
                          ↘
                           infrastructure

strategies → domain + indicators (+ common/core)
risk       → domain (+ common/core)
execution  → domain (+ common/core)
market     → domain (+ common/core)   # 仅接入与转换；不落库

infrastructure → domain + core + common
services       → domain + (strategies/risk/execution/market) + infrastructure
```

### 硬边界检查（可执行标准）
- **strategies**：不得出现 `sqlx`/`redis`/`reqwest`/`okx`/`tokio_tungstenite`；不得引用 `market`/`infrastructure`
- **market**：不得出现 `sqlx::FromRow`；不得出现 `*Repository`；不得引用 `infrastructure`
- **services**：可以引用 `infrastructure`，但字段类型必须是 `domain` 的 trait（端口），禁止持有具体实现类型

### 依赖倒置原则 (DIP)
**services层访问infrastructure的正确方式**:
- ✅ **声明依赖**: 使用 `domain::traits::XxxRepository` (接口)
- ✅ **注入实现**: 运行时注入 `infrastructure::repositories::SqlxXxxRepository` (具体实现)
- ✅ **类型声明**: 字段类型为 `Box<dyn XxxRepository>` 或 `Arc<dyn XxxRepository>`
- ❌ **禁止**: 直接声明 `SqlxXxxRepository` 类型

**为什么这样设计**:
- domain层定义接口（稳定，不变化）
- infrastructure层实现接口（可替换，如MySQL → PostgreSQL）
- services层依赖接口（解耦，易于测试）

### 禁止依赖
- ❌ domain 不能依赖任何业务包或基础设施
- ❌ infrastructure 不能依赖 strategies/risk/execution/services/orchestration
- ❌ indicators 不能依赖 strategies/risk/execution/services
- ❌ strategies 不能依赖 risk/execution/market/infrastructure/services/orchestration
- ❌ risk 不能依赖 strategies/execution/market/infrastructure/services/orchestration
- ❌ execution 不能依赖 strategies/risk/market/infrastructure/services/orchestration
- ❌ market 不能依赖 services/orchestration/infrastructure（防止“接入层”反向污染）
- ❌ services 不能把 `infrastructure` 的具体类型当成字段类型（字段应为 domain trait：`Arc<dyn Xxx>`/`Box<dyn Xxx>`）

---

## 📝 命名规范

### 文件命名
- 实体: `xxx.rs` (domain) 或 `xxx_entity.rs` (infrastructure)
- 值对象: `xxx.rs`
- Repository: `xxx_repository.rs`
- Service: `xxx_service.rs`
- 策略: `xxx_executor.rs` 或 `xxx_strategy.rs`
- 任务: `xxx_job.rs`
- 指标: `xxx.rs` 或 `xxx_indicator.rs`

### 类型命名
- 实体: `Xxx` (如 Order, Candle)
- 值对象: `Xxx` (如 Price, Volume)
- Repository: `SqlxXxxRepository`
- Service: `XxxService`
- 错误: `XxxError`

---

## ✅ 快速检查清单

新增或者修改代码前检查:
- [ ] 代码放在正确的包中？
- [ ] 文件放在正确的目录中？
- [ ] 没有违反依赖规则？
- [ ] 命名符合规范？
- [ ] domain包没有外部框架依赖？
- [ ] infrastructure实现了domain接口？

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fairwic)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fairwic)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
