## work-product

> Work 专家工作台（screens/Hermes）模块约束 — SMC Copilot 内 Work 域，非 WorkBuddy 品牌


# Work Product Line（Work 域）

## 产品身份

- **对外产品名**：SMC Copilot（不变）
- **Work 域**：`src/renderer/src/screens/Hermes/` — Work 专家工作台（原 Local Hermes 产品概念）
- **WorkBuddy（腾讯）**：仅 UX/流程参考；**禁止**作为代码标识符、Preload 全局名或对外品牌

## Renderer 分层

```text
shell/     → 布局装配，不直接调 API
registry/  → 页面元数据，无业务逻辑
pages/     → 页面编排，调 feature hooks
features/  → 数据获取、映射、校验、动作
api/       → workApi.ts，封装 window.hermesExperts / window.hermesAPI
model/     → Work* 域类型
components/→ 纯展示，不调 API
```

## 硬性规则

1. **禁止** pages 直接调用 `window.hermesExperts` 或 `window.hermesAPI`
2. **禁止** components 直接调 API 或读复杂 Context 业务状态
3. **禁止** Renderer `fetch` nodeskclaw 或持有 token
4. **禁止** 在 pages 内拼 JSON-RPC 原始 payload
5. v1.3 **不新增** `window.work` Preload；使用 `api/workApi.ts`
6. Run / Expert / Artifact 类型从 `model/` 导入，使用 `Work*` 前缀

## 新增能力流程

1. Main 实现（已有 `hermes-experts/*` 优先复用）
2. Preload `hermes-experts-api.ts`（v1.3 不新增全局对象）
3. `api/workApi.ts` 语义封装 + `model/*` 映射
4. `features/*` hook
5. `pages/*` 消费

## AI Coding 任务粒度

每次只改一个闭环（registry / sidebar / 单页 / workApi 子域），改后运行 `npm run typecheck`。

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
