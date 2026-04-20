## mindpocket

> 这是一个基于 Turborepo 的 monorepo 项目

# AGENT

这是一个基于 Turborepo 的 monorepo 项目

## Architecture

### Tech Stack

**Web 应用:**
- **框架**: Next.js 16 (App Router)
- **UI**: Radix UI + Tailwind CSS 4
- **认证**: Better Auth (基于 email/password)
- **数据库**: PostgreSQL + Drizzle ORM (Neon Serverless)
- **AI**: Vercel AI SDK + OpenAI
- **状态管理**: Zustand
- **动画**: Motion (Framer Motion)
- **其他**: React Native Web (跨平台组件共享)

**Native 应用:**
- **框架**: Expo + React Native
- **路由**: Expo Router
- **UI**: HeroUI Native + Tailwind (uniwind)
- **导航**: React Navigation

### 认证架构

使用 Better Auth 实现认证系统：
- **服务端**: `apps/web/lib/auth.ts` - Better Auth 配置
- **客户端**: `apps/web/lib/auth-client.ts` - 客户端 SDK
- **API 路由**: `apps/web/app/api/auth/[...all]/route.ts` - 认证端点
- **数据库 Schema**: `apps/web/db/schema/auth.ts` - 用户和账户表

### 数据库架构

- **ORM**: Drizzle ORM
- **数据库**: PostgreSQL (Neon Serverless)
- **配置文件**: `apps/web/drizzle.config.ts`
- **Schema 位置**: `apps/web/db/schema/`
- **迁移文件**: `apps/web/db/migrations/`
- **客户端**: `apps/web/db/client.ts`

环境变量需要在 `apps/web/.env.local` 中配置 `DATABASE_URL`。

### UI 组件

- **组件库**: 使用 shadcn/ui 风格的组件系统
- **配置文件**: `apps/web/components.json`
- **组件位置**: `apps/web/components/ui/`
- **自定义组件**: `apps/web/components/` (如 login-form, sidebar-left 等)

## Important Notes

### 包管理器

- 必须使用 **pnpm** (版本 10.9.0)
- 这是一个 workspace monorepo，使用 `workspace:*` 协议引用内部包

### 代码风格

- 使用 **Biome** 进行 lint 和格式化（不是 ESLint/Prettier）
- 使用 **Ultracite** 进行额外的代码检查

### 数据库工作流

1. 修改 schema 文件 (`apps/web/db/schema/`)
2. 运行 `pnpm db:generate` 生成迁移
3. 运行 `pnpm db:migrate` 应用迁移（生产环境）
4. 或运行 `pnpm db:push` 直接推送（开发环境）

### 创建新用户

由于认证系统需要密码哈希，不能直接在数据库中创建用户。使用提供的脚本：

```bash
cd apps/web
pnpm tsx scripts/create-user.ts your@email.com yourpassword "Your Name"
```

最新版 nextjs 的 middleware 改名 proxy.ts ，请注意区分


### Turbo 缓存

Turborepo 会缓存构建结果以加速后续构建。如果遇到缓存问题，可以：
- 删除 `.turbo` 目录
- 运行 `pnpm clean` 清理所有缓存

完成任务后，需要检查代码和格式化


# 注意

写代码要求有简单注释


完成一个任务后需要进行，代码检查和格式化
pnpm lint          # 使用 Biome 进行 lint
pnpm format        # 使用 Biome 格式化代码
pnpm check         # 使用 Ultracite 检查
pnpm fix           # 使用 Ultracite 修复

以确保代码质量。

做完后使用 chrome dev tool mcp 测试界面是否符合预期


## 参考
./docs/PROJECT.md

---
> Source: [jihe520/mindpocket](https://github.com/jihe520/mindpocket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
