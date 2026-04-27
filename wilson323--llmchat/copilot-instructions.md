## llmchat

> 接口契约演进与版本化

# API 契约演进

- 契约：请求/响应包含 `code`、`message`、`data`；共享类型在 [shared-types/src/index.ts](mdc:shared-types/src/index.ts)。
- 版本化：路径或 Header（如 `/api/v1`）；避免破坏性变更；弃用提供迁移指南与宽限期。
- 校验：后端入参校验；前端类型对齐；PR 必附变更说明。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wilson323) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
