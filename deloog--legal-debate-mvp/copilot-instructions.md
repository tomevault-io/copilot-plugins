## legal-debate-mvp

> 本文件在每次 AI 会话启动时自动加载，记录了项目的编码规范和长期改进要求。

# Claude Code 项目指南

本文件在每次 AI 会话启动时自动加载，记录了项目的编码规范和长期改进要求。

---

## 零、交流语言

**与用户的所有对话必须使用中文**，包括解释、说明、进度汇报等。代码注释和变量名保持英文。

---

## 一、项目概述

- **框架**：Next.js 16 App Router + TypeScript + Prisma + Tailwind CSS
- **认证**：JWT（`src/lib/auth/jwt.ts`）+ NextAuth（`src/lib/auth/auth-options.ts`）
- **日志**：统一使用 `import { logger } from '@/lib/logger'`，禁止 `console.*`
- **API 规范**：所有接口返回 `{ success: boolean, data?: T, error?: { code, message } }`

---

## 二、编码规范（每次修改代码时必须遵守）

### 2.1 禁止新增 `any` 类型

- **不允许**在新写的代码中使用 `any`，包括 `as any`
- 如果遇到 Prisma JSON 字段或第三方库类型不完整，使用 `unknown` + 类型守卫，或 `Record<string, unknown>`
- ESLint 规则：`@typescript-eslint/no-explicit-any: warn`，新代码中警告即需修复

### 2.2 顺手修复规则（机会性改进）

**每次修改某个文件时，如果该文件中存在以下问题，必须顺带修复：**

1. `any` 类型 → 替换为具体类型或 `unknown`
2. `eslint-disable` 注释（非必要的）→ 删除并修复根本原因
3. `console.log/warn/error` → 替换为 `logger.*`

> 这条规则的目的是防止技术债务永远积压。不要等到"专项重构"才处理，改哪个文件就顺手清理哪个文件。

### 2.3 已知技术债务清单

下列问题已记录，遇到时按"顺手修复规则"处理，**不需要专项处理**：

| 类型             | 数量（估计） | 说明                                                      |
| ---------------- | ------------ | --------------------------------------------------------- |
| `any` / `as any` | ~175 处      | 分布在 src/ 各处，logger.ts 的 any 是结构性必要，其余可改 |
| `next-auth` 版本 | 1 处         | 当前 v4.24.13，计划升级到 v5（需要单独迭代）              |

---

## 三、架构约定

### 3.1 API 路由认证

所有需要用户身份的 API 路由使用 JWT 验证模式：

```typescript
const authHeader = request.headers.get('authorization');
const token = extractTokenFromHeader(authHeader ?? '');
const tokenResult = verifyToken(token ?? '');
if (!tokenResult.valid || !tokenResult.payload) {
  return NextResponse.json(
    { success: false, message: '未授权' },
    { status: 401 }
  );
}
const { userId } = tokenResult.payload;
```

### 3.2 前端数据获取

- 使用原生 `fetch` + `useState(loading/error/data)` 模式
- 不引入 SWR 或 React Query（保持简单）
- 错误显示用 `setError(data.message || data.error?.message || '操作失败')`

### 3.3 数据库访问

- 统一从 `@/lib/db/prisma` 导入 `prisma` 实例
- Decimal 字段转换：`Number(prisma_decimal_value)`
- 日期字段序列化：`.toISOString()`

---

## 四、目录结构说明

```
src/app/api/          # API 路由（Next.js Route Handlers）
src/app/*/page.tsx    # 前端页面（'use client'）
src/lib/              # 业务逻辑、服务层
src/types/            # 共享类型定义
prisma/schema.prisma  # 数据库模型
src/lib/crawler/archive/  # 已废弃的爬虫，不参与运行时，忽略其中的 TODO
```

---

## 五、测试

- 单元测试：`src/__tests__/`，使用 Jest
- E2E：`src/__tests__/e2e/`，使用 Playwright
- 运行：`npm test`（单元）/ `npm run test:e2e`（E2E）
- TypeScript 编译检查：`npx tsc --noEmit`（每次完成修改后运行）

---
> Source: [deloog/legal-debate-mvp](https://github.com/deloog/legal-debate-mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
