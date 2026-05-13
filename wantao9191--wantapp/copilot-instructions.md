## database

> 数据库开发规范


# 数据库开发规范

## 数据库技术栈

- **数据库**: PostgreSQL
- **ORM**: Drizzle ORM
- **迁移**: Drizzle Kit
- **连接**: postgres 驱动

## 数据库配置

数据库配置在 [src/db/index.ts](mdc:src/db/index.ts) 中：

- 使用环境变量配置连接
- 支持开发和生产环境
- 连接池配置优化

## Schema 定义

数据库模式定义在 [src/db/schema.ts](mdc:src/db/schema.ts)：

- 使用 Drizzle 的 schema 语法
- 定义表结构和关系
- 包含索引和约束

## 迁移管理

- 迁移文件在 `drizzle/` 目录
- 使用 `pnpm db:generate` 生成迁移
- 使用 `pnpm db:migrate` 应用迁移
- 使用 `pnpm db:push` 直接推送 schema

## 数据库脚本

项目包含多个数据库管理脚本：

- `scripts/create-database.ts` - 创建数据库
- `scripts/seed.ts` - 种子数据
- `scripts/insert-menus.ts` - 插入菜单数据

## 查询优化

- 使用 Drizzle 的类型安全查询
- 合理使用索引
- 避免 N+1 查询问题
- 使用事务处理复杂操作

## 数据验证

- 使用 Zod 进行数据验证
- 在 API 层进行输入验证
- 数据库层进行约束验证

---
> Source: [wantao9191/wantapp](https://github.com/wantao9191/wantapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
