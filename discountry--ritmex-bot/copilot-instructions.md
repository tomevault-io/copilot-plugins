## ritmex-bot

> **必须使用 Bun** — 本项目使用 Bun 作为包管理器和运行时。所有命令都必须用 bun 执行，**不要使用 npm、yarn 或 npx**。

# Repository Guidelines

## Package Manager

**必须使用 Bun** — 本项目使用 Bun 作为包管理器和运行时。所有命令都必须用 bun 执行，**不要使用 npm、yarn 或 npx**。

## Project Structure

入口 `index.ts` 在仓库根目录，用于 CLI 启动。所有生产代码在 `src/` 下：

```
src/
├── config.ts              # 运行时配置（GridConfig、MakerPointsConfig 等）
├── runtime-errors.ts      # 全局错误处理
├── index.tsx              # Ink 渲染入口
├── cli/                   # CLI 参数解析与策略启动
│   ├── args.ts
│   ├── command-executor.ts
│   ├── command-parser.ts
│   ├── command-types.ts
│   └── strategy-runner.ts
├── core/                  # 订单协调 + 共享库
│   ├── order-coordinator.ts   # 限价单/市价单统一下单（锁、去重、速率控制）
│   └── lib/
│       ├── order-plan.ts
│       ├── orders.ts
│       └── rate-limit.ts
├── exchanges/             # 交易所适配器（每个子目录含 adapter/gateway/order）
│   ├── adapter.ts         # ExchangeAdapter 接口
│   ├── adapter-utils.ts   # 适配器工具函数
│   ├── types.ts           # CreateOrderParams、Order、AccountSnapshot 等公共类型
│   ├── order-schema.ts    # BaseOrderIntent、LimitOrderIntent 等
│   ├── order-handlers.ts  # 通用 order handler 工厂
│   ├── order-router.ts    # 按交易所分发订单处理
│   ├── create-adapter.ts  # 工厂函数
│   ├── resolve-from-env.ts
│   ├── dry-run-adapter.ts # 模拟适配器
│   ├── aster/             # Aster 交易所
│   ├── backpack/          # Backpack 交易所
│   ├── binance/           # Binance 交易所（CCXT）
│   ├── grvt/              # GRVT 交易所
│   ├── lighter/           # Lighter 交易所（含签名/nonce/字节处理）
│   ├── nado/              # Nado 交易所
│   ├── paradex/           # Paradex 交易所
│   └── standx/            # StandX 交易所
├── strategy/              # 所有策略引擎
│   ├── maker-engine.ts        # Maker 做市策略
│   ├── maker-points-engine.ts # MakerPoints 积分做市策略
│   ├── maker-points-logic.ts  # MakerPoints 纯逻辑（可单独测试）
│   ├── offset-maker-engine.ts # 偏移做市策略
│   ├── trend-engine.ts        # 趋势跟踪策略
│   ├── grid-engine.ts         # 网格交易策略（LevelState + clientOrderId 恢复）
│   ├── basis-arb-engine.ts    # 基差套利策略
│   ├── swing-engine.ts        # 波段交易策略
│   ├── swing-logic.ts         # 波段纯逻辑
│   ├── guardian-engine.ts     # 监控守护策略
│   ├── liquidity-maker-engine.ts # 流动性做市策略
│   └── common/                # 策略共享辅助
│       ├── binance-depth.ts       # Binance 深度分析
│       ├── binance-rsi.ts         # Binance RSI 指标
│       ├── event-emitter.ts       # 策略事件发射器
│       ├── grid-storage.ts        # 网格状态磁盘持久化
│       ├── session-volume.ts      # 会话成交量统计
│       └── subscriptions.ts       # WebSocket 安全订阅
├── ui/                    # Ink 仪表盘（每个策略独立 App）
│   ├── App.tsx
│   ├── GridApp.tsx
│   ├── MakerApp.tsx
│   ├── MakerPointsApp.tsx
│   ├── OffsetMakerApp.tsx
│   ├── TrendApp.tsx
│   ├── BasisApp.tsx
│   ├── SwingApp.tsx
│   ├── GuardianApp.tsx
│   ├── LiquidityMakerApp.tsx
│   └── components/
├── logging/               # 交易日志
│   └── trade-log.ts
├── notifications/         # Telegram 通知
│   ├── index.ts
│   ├── telegram.ts
│   └── types.ts
├── i18n/                  # 国际化
│   └── index.ts
└── utils/                 # 跨领域工具
    ├── math.ts, format.ts, price.ts, depth.ts
    ├── errors.ts, risk.ts, pnl.ts
    ├── strategy.ts, order-status.ts, security.ts
    ├── copyright.ts
    └── standx-token-expiry.ts
```

辅助目录：
- `tests/` — Vitest 测试套件（大部分测试在此，少量与逻辑共处于 `src/`）
- `docs/` — 交易所 API 参考文档
- `scripts/` — 一次性脚本（密钥派生等）
- `skills/` — AI 助手技能配置

## Commands

```bash
bun install                  # 安装依赖
bun run dev                  # 启动 CLI 菜单（等同于 bun run index.ts）
bun run test                 # 执行全部测试（bun x vitest run）
bun x vitest --watch         # 增量测试
bun x tsc --noEmit           # 类型检查（无 typecheck 脚本，直接用 tsc）
bun run lint                 # oxlint 检查
bun run lint:fix             # oxlint 自动修复
```

⚠️ **不要用 `bun test`** — 那是 Bun 内置 runner，不支持 `vi.resetModules()`、`vi.waitFor()` 等 Vitest API。必须用 `bun run test`。

## Coding Style

- 现代 TypeScript + ES modules，严格模式（`strict: true`）
- 两空格缩进，imports 按 external → internal 排序
- 变量/函数用 `camelCase`，类/枚举用 `PascalCase`
- 新策略放 `src/strategy/`，策略共享工具放 `src/strategy/common/`
- 跨领域工具放 `src/utils/`
- 注释只写非显而易见的交易逻辑
- JSX 用于 Ink UI 组件（`react-jsx`）

## Testing

Vitest 驱动所有测试。测试文件两种位置：
- `tests/<feature>.test.ts` — 大部分测试
- `src/<module>/<feature>.test.ts` — 纯逻辑单元测试（如 `maker-points-logic.test.ts`、`swing-logic.test.ts`）

策略测试应覆盖：订单生命周期、风控守卫、WebSocket 边界情况、状态恢复。

## Commit Guidelines

Conventional Commits（`feat:` / `fix:` / `refactor:` / `test:`），每次提交作用域限单个模块或策略。

## Environment & Secrets

复制 `.env.example` 为 `.env` 填入 API 密钥。**不要读取、打印或提交 `.env` 和任何密钥。** 泄露立即轮换。

## Safety

- 永远不要读取、运行、打印或访问 `.env` 或任何 secrets
- 代码中遇到项目依赖时，先查版本再查文档，避免过时知识导致错误

---
> Source: [discountry/ritmex-bot](https://github.com/discountry/ritmex-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
