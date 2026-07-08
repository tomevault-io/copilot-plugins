## ai-admin-pro

> > React 18 + TypeScript 5 + @dalydb/sdesign + antd 5 + Zustand + Rsbuild

﻿# AI Frontend App

> React 18 + TypeScript 5 + @dalydb/sdesign + antd 5 + Zustand + Rsbuild
>
> **这是 AI 进入本项目的唯一入口。**

### 设计目标

**让 AI 在运行时要做的决策降到最低。** 一切规约、模板、闸门、配方为此服务。

三招落地：

1. **决策预编译** — 能路由表匹配的不让推理，能步骤表固化的不给自由，能模板填空的不让生成
2. **错题集驱动** — 翻车了才加规则，没翻的不管。21 行错题集 > 全量知识库
3. **复杂度外移** — 能用工具侧解决的复杂度不给 AI。约束 > 自由，禁止 > 建议

> 完整论述 → `.ai/core/ai-engineering-principles.md`

### 核心原则

⑨ 人是最终责任者 > ① 复杂度不灭 > ③ 边界是宪法 > ② 为修改而设计 > ④ 假设一切会炸 > ⑦ 能观测才存在 > ⑧ 技术债是债务 > ⑤ 先跑通再跑对再跑快 > ⑥ 代码是给人读的

---

## 零、执行前检查清单（必读，不可跳过）

任何代码生成之前，按顺序读：

| #   | 文件                                               | 读取时机                                     |
| --- | -------------------------------------------------- | -------------------------------------------- |
| 1   | `.ai/project-brief.md`                             | 每次新会话                                   |
| 2   | `.ai/core/routing-strategy.md`                     | 每次新会话                                   |
| 3   | `.ai/conventions/task-splitting.md`                | 拆 Task 前                                   |
| 4   | `.ai/conventions/lanes.md`（如果 §一 匹配了 Lane） | 生成前                                       |
| 5   | `.ai/templates/{匹配的模板}.md`                    | 生成前                                       |
| 6   | `.ai/sdesign/components/{用到的 S 组件}.md`        | 生成前（首次生成时必读，不受防循环规则限制） |
| 7   | `.ai/pitfalls/index.md`                            | 生成前                                       |
| 8   | `.ai/recipes/index.md`（如果有配方匹配表）         | 后置任务时                                   |

> 禁止跳过。禁止凭记忆写代码。

**防死循环**（每步执行前检查）：

| 规则 | 说明                                                        |
| ---- | ----------------------------------------------------------- |
| R1   | 同一文件编辑 ≥3 次仍有错误 → **停止，报告用户**，不要继续改 |
| R2   | 连续 3 轮 verify 同类错误不减少 → **强制停止，报告状态**    |

> pnpm verify 最多跑 3 轮（见 §三）。

---

## 一、场景匹配（不推理，纯匹配）

| 用户意图                             | 走什么                                            |
| ------------------------------------ | ------------------------------------------------- | ---------------------- | -------------------------------------------------------------------- |
| 小修改（改字段/修bug/调样式，≤20行） | → 修改路径，见 `.ai/conventions/modify-paths.md`  |
| 新建/新模块 + 列表/CRUD/增删改查     | → CRUD Lane（模板填空，不生成 PRD）               |
| 大屏 + 先出蓝图/设计方案/规划        | → 大屏蓝图 Lane（生成蓝图 → 自动续接大屏 Lane）\n | 大屏 + 生成代码/写页面 | → 大屏 Lane（先评估复杂度：简单走轻量路径跳过蓝图，复杂走蓝图 Lane） |
| 新建 + 详情（含多Tab）               | → 多Tab详情 Lane（增量追加）                      |
| 无法匹配任何场景                     | → 非标 Lane（PRD 兜底）                           |

> Lane 详细流程 → `.ai/conventions/lanes.md`

---

## 二、硬约束（所有 Lane 通用）

### 组件替换

| 禁止              | 必须用                           |
| ----------------- | -------------------------------- |
| antd Table        | STable / SProTable               |
| antd Form         | SForm / SForm.Search             |
| antd Button       | SButton                          |
| antd Descriptions | SDetail                          |
| echarts-for-react | EChartsBase（`src/components/`） |

> 组件详细 API → `.ai/sdesign/components/{Name}.md`

### 导入规则

| 规则       | 正确写法                                   |
| ---------- | ------------------------------------------ |
| HTTP 请求  | `createRequest` from `src/plugins/request` |
| 类型导入   | `import type { X }`                        |
| 路径别名   | `src/` 禁止 `../`                          |
| API 命名   | `getListByGet` / `createByPost` 等         |
| 未使用参数 | 加 `_` 前缀                                |

### 类型

- 零 `any`（保底 `Record<string, unknown>`）
- 全局类型 `PageData<T>` / `PageQuery` 在 `src/types/index.ts`，禁止重复定义
- 模块 types.ts 只定义：`{Entity}` + `{Entity}Query extends PageQuery` + `{Entity}FormData`
- TS 详细规约 → `.ai/conventions/typescript-conventions.md`

### 闸门

- 每个 Task 遵守输出锁 + 两层闸门（全局 G1-G15 + Task 类型闸门）→ `.ai/conventions/task-gates.md`
- `pnpm verify` 报错时只修本 Task 输出锁内的文件。范围外跳过。
- ≤10 新文件 / ≤10 修改文件。超出先列清单确认。

### 风险操作

| 自由操作                       | 需确认                                             | 禁止                                                             |
| ------------------------------ | -------------------------------------------------- | ---------------------------------------------------------------- |
| 读文件、搜索、verify、新建模块 | 删文件、改 eslint/rsbuild/tsconfig、改 `.ai/` 规范 | 改输出锁外文件、自行安装依赖、改 plugins/ 或 router/（未经同意） |

---

## 三、生成流程

```
读需求 → 匹配场景（§一）→ 拆 Task（先读 `.ai/conventions/task-splitting.md`）→ 逐 Task 执行：

  1-3. 按 §零 检查清单读取所需文件（⚠️ 不要跳过）
  4. 填空生成
  4.1 后置任务 — 扫 .ai/recipes/index.md，命中则执行配方
  5. pnpm verify（最多 3 轮）
  6. pnpm verify 通过后，标记 Task ✅ + 回答复盘三问（禁止跳过）：
     Q1: verify 报过什么错？直接复制 terminal 输出，没报错写"0 error"
     Q2: 报错的根因是什么？（缺类型定义 / 组件 API 用错 / 导入缺失 / 格式不齐 / 其他）
     Q3: 需要沉淀到错题集吗？（无 / 有: 错误签名 + 修复方法）
     → 三问答完才能进下一个 Task
```

---

## 四、关键文件索引

| 需要               | 文件                                        |
| ------------------ | ------------------------------------------- |
| 认知底座           | `.ai/project-brief.md`                      |
| 路由策略 + 配方    | `.ai/core/routing-strategy.md`              |
| Lane 详细流程      | `.ai/conventions/lanes.md`                  |
| 修改路径           | `.ai/conventions/modify-paths.md`           |
| Task 闸门 + 输出锁 | `.ai/conventions/task-gates.md`             |
| API 规范           | `.ai/conventions/api-conventions.md`        |
| TS 规约            | `.ai/conventions/typescript-conventions.md` |
| Task 拆分规约      | `.ai/conventions/task-splitting.md`         |
| 配方闸门           | `.ai/conventions/recipe-conventions.md`     |
| 代码模板           | `.ai/templates/`                            |
| 错题集             | `.ai/pitfalls/index.md`                     |

---
> Source: [daly-dai/ai-admin-pro](https://github.com/daly-dai/ai-admin-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
