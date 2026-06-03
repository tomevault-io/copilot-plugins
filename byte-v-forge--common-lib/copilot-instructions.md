## common-lib

> 本仓承载 byte-v-forge 跨仓平台通用库。

# AGENTS.md

本仓承载 byte-v-forge 跨仓平台通用库。
- 公开契约位于 `proto/byte/v/forge/contracts/`，承载跨仓稳定公开建模和 gRPC service；Go 生成物位于 `gen/go/byte/v/forge/contracts/`，前端 TypeScript 生成物位于 `ui/src/proto/byte/v/forge/contracts/`；Python SDK 通过 `scripts/generate-python-proto.sh` 按需生成，不手写。
- 严禁把 internal/private/provider raw shape、业务状态机、业务页面或 service 私有配置迁入 `common-lib`；这些模型继续留在拥有方子仓。

- Go 通用库只放无业务语义的基础设施 helper，例如 gRPC health、日志/配置等稳定小边界。
- UI 通用库位于 `ui/`，承载 shadcn/Radix 基础组件、dashboard uikit、通用数据驱动组件、格式化/剪贴板/toast 等无业务/provider 语义能力。
- 禁止放入 GPT、Mailbox、SMS、Proxy、Browser Automation 等业务/provider 分支、页面、数据请求或状态机。
- 跨仓使用必须通过 Go module、npm package 或部署期包边界消费，不直接复制实现。
- `ui/src/components/ui/` 下 shadcn/Radix 生成组件按第三方模板维护；优先用官方 `shadcn` CLI 更新，不手写等价基础控件。
- Linter 检查必须达到 0 error / 0 warning；禁止通过修改或放宽 linter 配置、降低规则级别、删除规则、添加 ignore/disable/nolint/ts-ignore/eslint-disable/biome-ignore/prettier-ignore 等方式绕过问题，只能按 linter 规则修复源码、类型、格式或依赖边界。

---
> Source: [byte-v-forge/common-lib](https://github.com/byte-v-forge/common-lib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
