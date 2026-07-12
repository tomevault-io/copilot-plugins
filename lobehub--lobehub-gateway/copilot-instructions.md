## lobehub-gateway

> - 面向开发者和用户描述最终行为、接口、配置与约束；仅在 changelog 或迁移指南中记录演变过程。

@README.md

## 开发约定

- 面向开发者和用户描述最终行为、接口、配置与约束；仅在 changelog 或迁移指南中记录演变过程。
- 默认遵循 README 固定的协议基线；任务指定 LobeHub tag 或 commit 时以该版本为准，只有明确要求时才跟踪 `main` 或 `canary`。

## 上游参考

协议核对时建议参考：

- [LobeHub](https://github.com/lobehub/lobehub)：应用侧协议，重点关注 `packages/device-gateway-client`、`packages/agent-gateway-client` 和 `src/server/agent-hono/handlers`。
- [Device Gateway](https://github.com/lobehub-biz/device-gateway)：Device Gateway 服务实现。
- [Agent Gateway](https://github.com/lobehub-biz/agent-gateway)：Agent Gateway 服务实现。

需要查看完整的上游服务实现时，可将对应仓库 clone 到项目根目录下的 `device-gateway/` 或 `agent-gateway/`；这两个目录仅用于参考，不属于本项目源码。

---
> Source: [lobehub/lobehub-gateway](https://github.com/lobehub/lobehub-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
