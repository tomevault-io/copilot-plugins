## ly-fullstack

> This file provides guidance to AI coding agents (Codex / Claude Code 等) when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents (Codex / Claude Code 等) when working with code in this repository.

---

## 关于本文件

这是一份**个人编码 DNA**，记录编程思想与核心原则。
具体编码规范见 `.rules/`，项目入口说明见 `README.md`，阶段性文档见 `docs/`。

---

## 一、编程思想

### 1. 先思考，再动手

**不要假设。不要隐藏困惑。暴露权衡。**

- 动手前，明确说出你的假设。不确定就问。
- 存在多种理解时，全部列出 — 不要默默选一个。
- 有更简单的方案就说出来。该推回时推回。
- 遇到不清楚的，停下来。说出什么让你困惑。问。

### 2. 简洁至上

**最少的代码解决问题。不做投机。**

- 不加需求之外的功能。
- 单次使用的代码不做抽象。
- 没人要求的"灵活性"和"可配置性"不加。
- 不为不可能发生的场景写错误处理。
- 200 行能用 50 行实现，重写。

自问：**"高级工程师会说这过度复杂了吗？"** 如果会，简化。

#### 最小实现决策梯子

参考 [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) 的核心思想：AI 写代码前，先像一个懒但强的资深工程师一样判断这段代码是否值得存在。

在动手实现前，按顺序问：

1. **这个东西真的需要存在吗？** 不需要就不写。
2. **代码库里已经有了吗？** 有就复用，不重写。
3. **标准库能解决吗？** 能就用标准库。
4. **浏览器 / 平台原生能力能解决吗？** 能就用原生能力。
5. **现有依赖能解决吗？** 能就用已安装依赖，不新增依赖。
6. **一行或少量代码能解决吗？** 能就不要造抽象。
7. **最后才写最小可行实现。**

这不是追求代码越短越好，而是追求**必要且刚好够用**。安全、权限、数据丢失、错误处理、可访问性、类型正确性和业务正确性不属于可以被压缩掉的内容。

### 3. 外科手术式修改

**只动必须动的。只清理自己的痕迹。**

修改已有代码时：

- 不"顺手改进"相邻的代码、注释或格式。
- 不重构没坏的东西。
- 匹配已有风格，即使你会用不同的写法。
- 发现不相关的死代码，提一嘴 — 不要删它。

当你的修改产生了孤立代码时：

- 移除因**你的修改**而变得多余的 import/变量/函数。
- 不移除改动之前就存在的死代码，除非被要求。

**检验标准：每一行改动都应该直接追溯到用户的需求。**

### 4. 目标驱动执行

**定义成功标准。循环验证直到达成。**

把任务转化为可验证的目标：

- "加验证" → "写无效输入的测试，然后让它通过"
- "修 bug" → "写复现测试，然后让它通过"
- "重构 X" → "确保重构前后测试都通过"

多步任务，先列简要计划：

```
1. [步骤] → 验证：[检查项]
2. [步骤] → 验证：[检查项]
3. [步骤] → 验证：[检查项]
```

强成功标准让你能独立循环。弱标准（"搞定就行"）需要反复确认。

---

## 二、核心原则

### 架构决策框架

做技术选型和架构决策时，按以下优先级排序：

1. **正确性** > 性能 > 优雅
2. **可读性** > 可复用性 > 灵活性
3. **显式** > 隐式 > 魔法
4. **组合** > 继承 > 混入

### 我的编码信条

- **组件薄、逻辑厚** — 视图层只负责渲染和交互，业务逻辑抽到 hooks/helper
- **barrel 导出统一** — 每个模块目录都有 `index.ts` 聚合导出
- **类型安全链** — 类型定义一次，全链路共享，不在多处重复定义
- **服务实例隔离** — 不同后端服务独立 Axios 实例和拦截器，互不干扰
- **注释解释 WHY，不解释 WHAT** — 代码自解释的部分不写注释
- **能推断的不标注** — TypeScript 类型标注只在推断不出时才写

### 什么是绝对不妥协的

- 不使用 `any`（后端 DTO 场景除外）
- 不在组件内写业务逻辑超过 20 行 — 超过就抽 hook/helper
- 不在一个文件里定义多个组件
- 不用 Options API（Vue 3 项目）

### 编码完成自查

**每次完成一个功能模块后，必须从以下两个维度深度审视：**

1. **内存泄漏** — 事件监听是否在 `onUnmounted` 清除？定时器/订阅是否销毁？闭包是否持有大对象？SSE/WebSocket 连接是否正确关闭？
2. **容错处理** — 网络异常是否有兜底？空数据/null 是否有防御？用户快速操作（连点、切页面）是否有保护？异步竞态是否处理？

详见 `.rules/code-review.md` 完整自查清单。

---

## 三、技术栈

| 领域     | 选型                             | 备注                                                     |
| -------- | -------------------------------- | -------------------------------------------------------- |
| 仓库形态 | pnpm workspace + Turborepo       | `apps/*` + `packages/*` monorepo                         |
| 包管理   | pnpm 11.x                        | Node.js >= 22，根目录通过 `packageManager` 固定          |
| 管理后台 | Rsbuild 2 + Vue 3 + Element Plus | `apps/admin`，构建模块维护在 `apps/admin/build`          |
| 后台样式 | SCSS                             | `apps/admin/src/assets/styles` 维护变量、mixin、入口样式 |
| 管理 API | NestJS 11 + Fastify              | `apps/admin-api`，本地端口 3000；管理端独立认证边界      |
| 默认 API | NestJS 11 + Fastify              | `apps/api`，本地端口 3001；提供健康检查与公共读取能力    |
| 服务模板 | NestJS 11 + Fastify              | `scripts/templates/server`，按需生成独立服务             |
| 数据库   | PostgreSQL 17 + Prisma 7         | `packages/database` 是 Schema、迁移和 Client 唯一真相源  |
| 跨端共享 | TypeScript 手动维护              | `packages/shared`，统一存放共享类型与无 UI 框架通用工具  |
| 图表能力 | ECharts 6                        | `packages/charts`，按需注册无框架图表能力与公共类型      |
| 语言     | TypeScript 6.0.2                 | 当前 `6.0.2`，避免 `vue-tsc` 与 TypeScript 7 不兼容      |
| 质量     | ESLint 10 + Prettier + Husky     | lint-staged + commitlint                                 |
| 状态管理 | Pinia                            | `apps/admin` 已完成主题与认证会话状态及持久化边界        |
| 测试     | Rstest                           | `@rstest/core`，测试文件与源码同目录 `*.test.ts`         |

### 硬性架构边界

- **apps 不能被 packages 反向依赖**：依赖方向只能是 `apps → packages`；`packages/shared` 不得 import 任何 app 内代码。
- **服务端只使用 shared 类型入口**：所有 NestJS 服务只能从 `@repo/shared/types` 导入共享类型，不得依赖 `@repo/shared` 根入口或 `@repo/shared/utils` 中的浏览器工具。
- **charts 不承载业务组件**：`packages/charts` 只维护 ECharts 注册、初始化和公共类型；具体图表配置、数据转换、Vue 组件与页面交互留在应用内。
- **database 是服务端专用包**：`packages/database` 统一维护 Prisma Schema、migration、生成 Client 和数据库类型；不得依赖 NestJS，也不得被 admin/web 等浏览器应用导入。
- **Prisma 类型不得直通前端**：`packages/database/generated/prisma` 只在服务端消费；跨端传输的类型必须显式映射为 `packages/shared/src/types` 中的安全类型，再被前端引用。
- **外部 UI 项目只能作为参考**：曾参考的 `art-design-pro`（MIT）只用于借鉴视觉语言，不是本仓库运行依赖；禁止引入它的 Vite 构建、Tailwind、Router/Store 架构与业务组件。设计真相源是本仓库现有实现与 `docs/admin-design-system.md`。
- **服务认证边界不得混用**：`admin-api` 当前采用管理端 JWT 与五表 RBAC；默认 `api` 不包含终端用户认证，后续必须按真实 C 端业务独立设计，不得跨应用共享 Guard、会话或业务模块。
- **应用注册表是本地运行真相源**：应用分类、路径、包名、本地端口与健康检查统一维护在 `workspace.config.json`；应用源码不得另设本地默认端口。
- **只生成服务端应用**：新的 NestJS 服务使用 `pnpm new:server` 创建；主站技术栈可能是 Nuxt、Next.js 或其他方案，不提供统一 Web 生成器。

---

## 四、规范索引

具体编码规范按技术栈拆分在 `.rules/` 目录。执行任务前按下表路由读取：

| 任务类型          | 先读                                      | 说明                                         |
| ----------------- | ----------------------------------------- | -------------------------------------------- |
| 任何任务          | `.rules/comment-style.md`                 | 注释分组顺序                                 |
| 管理后台页面/CRUD | `.rules/admin.md`                         | 后台 CRUD 范本、分页、筛选、弹窗、表格规范   |
| 命名与目录        | `.rules/naming.md`、`.rules/directory.md` | 文件/变量/组件/常量命名表、monorepo 目录结构 |
| Vue 组件          | `.rules/vue3.md`                          | 组件结构、Props/Emits、样式策略、性能优化    |
| TypeScript        | `.rules/typescript.md`                    | 类型标注原则、类型组织、分层                 |
| 服务端模块        | `.rules/server.md`                        | 服务端类型归属、常量维护、Node ESM 导入      |
| 状态管理          | `.rules/pinia.md`                         | Store 写法、持久化策略                       |
| 请求层            | `.rules/axios.md`                         | 服务层封装、拦截器模式                       |
| 样式              | `.rules/style.md`                         | SCSS 组织、CSS 变量主题                      |
| 工程配置          | `.rules/engineering.md`                   | ESLint/Prettier/Git/路径别名                 |
| 错误处理          | `.rules/error-handling.md`                | 错误分层、容错模式、用户反馈                 |
| 提交前自查        | `.rules/code-review.md`                   | 内存泄漏、容错、完成度自查清单               |

项目入口说明维护在 `README.md`，专题文档维护在 `docs/` 目录，按需更新。

---

## 五、完成任务前的最小验证

从仓库根目录执行：

```bash
pnpm typecheck   # 全仓类型检查
pnpm lint        # ESLint
pnpm format:check
pnpm test        # Rstest 单元测试
pnpm build       # 构建全部产物
```

一次性执行全部检查：`pnpm check`。失败必须如实报告，禁止用脚本吞掉失败码。

---

**这些原则在起作用的标志：** diff 中没有不必要的改动，不因过度复杂而返工，澄清问题在实现之前而非犯错之后。

LY Fullstack 项目组

---
> Source: [liangy0323/ly-fullstack](https://github.com/liangy0323/ly-fullstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
