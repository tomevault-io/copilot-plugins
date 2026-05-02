## ipc-invoke

> 新增 IPC invoke 函数时的共享检查


# IPC Invoke 函数规则

## 新增 invoke 函数的判定流程

两端都需要 → 放 `packages/shared/src/ipc/types.ts` + 各端 re-export
仅一端需要 → 放该端 `src/ipc.ts`

## 当前已共享的 invoke 函数（禁止在各端重复定义）

- SecureStore: `setSecureValue`, `getSecureValue`, `deleteSecureValue`, `listSecureKeys`, `wipeSecureStore`
- MCP: `scanMcpServer`
- RAG: `readTextFile`
- Tailscale: `getTailscaleStatus`
- SSH Deploy: `deployRemoteConnect`, `deployRemoteCheckNode`, `deployRemoteInstallOpenclaw`, `deployRemoteOnboard`, `deployRemoteStartGateway`

## 各端 ipc.ts 的正确导入方式

```typescript
// ✅ 从 shared re-export
export { setSecureValue, getSecureValue } from "@clawno/shared/ipc/types";
export type { StepResult, SshArgs } from "@clawno/shared/ipc/types";

// ❌ 禁止本地重复定义
export const setSecureValue = (key: string, value: string) =>
  invoke<void>("set_secure_value", { key, value });
```

---
> Source: [clawno11/clawno11](https://github.com/clawno11/clawno11) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
