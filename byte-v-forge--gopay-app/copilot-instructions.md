## gopay-app

> 本仓是 `gopay-app` 独立业务服务仓，承载 GoPay App 账号、设备指纹、登录、注册、改绑、PIN、余额检查、OTP webhook、多用户状态管理，以及 Midtrans/GoPay linking/payment runtime。

# AGENTS.md

本仓是 `gopay-app` 独立业务服务仓，承载 GoPay App 账号、设备指纹、登录、注册、改绑、PIN、余额检查、OTP webhook、多用户状态管理，以及 Midtrans/GoPay linking/payment runtime。

- 本仓拥有 GoPay App 运行时、多用户状态、Midtrans + GoPay linking/payment runtime；GPT 侧只负责 ChatGPT checkout / Stripe 准备并传入已准备好的 `snap_token`。
- 跨仓调用通过 proto/gRPC、HTTP webhook、事件或部署配置完成；不得 import sibling repo 实现源码。
- `proto/` 是本仓服务契约真源；生成物不得手工修改。
- 多用户状态由本服务自己持有，热状态使用 Redis TTL；不要让 GPT 编排直接读写本服务状态存储。
- secret、token、OTP、cookie、支付凭据、代理凭据和会话材料不得输出到日志、指标、trace 或错误响应。
- 新增/调整运行时变量时同步 `deploy` 的 Compose、Helm values/templates 和示例值。
- Linter 检查必须达到 0 error / 0 warning；禁止通过修改或放宽 linter 配置、降低规则级别、删除规则、添加 ignore/disable/nolint/ts-ignore/eslint-disable/biome-ignore/prettier-ignore 等方式绕过问题，只能按 linter 规则修复源码、类型、格式或依赖边界。
- 不新增 CI/CD 配置。

---
> Source: [byte-v-forge/gopay-app](https://github.com/byte-v-forge/gopay-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
