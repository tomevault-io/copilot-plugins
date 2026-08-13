## frontend

> 编辑前端代码时参考——技术栈、风格、结构


# Frontend（按需参考）

- **技术栈**：Next.js 15 / React 19 / TypeScript 5；MUI 7、Zustand、TanStack React Query、React Hook Form、Zod、Axios。
- **风格**：2 空格、单引号、分号；PascalCase 组件/类型，camelCase 变量/Hooks，kebab-case 目录/文件；handle/ is/ has/ use 前缀。
- **结构**：`frontend/src/app`（App Router）、`components`、`hooks`、`lib`、`types`、`utils`。组件用 interface + 可选 onEdit；状态用 Zustand + React Query；日志见 `@/lib/logger`、`frontendLogger`。
- **性能**：useCallback/useMemo/React.memo/动态导入按需。安全：校验、JWT、RSA 签名。启动：`cd frontend && npm run dev`；需 npm install、.env.local、后端 8000。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
