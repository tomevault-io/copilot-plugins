## crypto-latency-probe-public

> 本仓库是固定范围的 Binance USD-M 延迟探针。

# AGENTS.md

本仓库是固定范围的 Binance USD-M 延迟探针。

## 边界

1. 固定测试 `BTCUSDT`、`btcusdt@bookTicker` 和 `POST /fapi/v1/order/test`，不要扩展成多交易所、策略或下单框架。
2. summary 统一使用 `p50_us`、`p95_us`、`p99_us`。Binance 原生毫秒时间戳必须显式乘除 `1_000`。
3. `order_test` 的本地计时使用微秒，签名参数 `timestamp` 仍使用毫秒。
4. 公开文档只记录匿名聚合结果，不写主机名、IP、供应商实例信息或私有部署映射。

## 验证

```bash
cargo fmt --check
cargo test --locked
cargo clippy --locked --all-targets -- -D warnings
cargo build --release --locked
docker build -t crypto-latency-probe .
docker run --rm crypto-latency-probe --help
```

---
> Source: [learnerLj/crypto-latency-probe-public](https://github.com/learnerLj/crypto-latency-probe-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
