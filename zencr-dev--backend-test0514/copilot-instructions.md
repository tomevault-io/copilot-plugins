## backend-test0514

> - 严格遵循 `schema.prisma` 定义规范，确保字段类型、关系、默认值和约束与数据库表结构设计一致。

# Prisma ORM使用规范

## 1. Schema 定义
- 严格遵循 `schema.prisma` 定义规范，确保字段类型、关系、默认值和约束与数据库表结构设计一致。
- 使用 `@@map("table_name")` 和 `@map("column_name")` 明确映射数据库表名和列名。
- 合理使用 `@@index` 定义索引，优化查询性能。

## 2. Prisma Client 使用
- 创建并使用 `PrismaService` 作为单例，集中管理Prisma Client实例，确保连接池的有效利用。
- 在服务层（Services）中注入 `PrismaService` 进行数据库操作。
- 避免在控制器（Controllers）中直接操作Prisma Client。

## 3. 事务管理
- 对于需要保证原子性的操作（例如订单创建涉及多表更新），务必使用Prisma的 `$transaction` API。
- 确保事务中的所有操作成功或全部回滚，保持数据一致性。

## 4. 查询最佳实践
- 使用 `select` 和 `include` 语句精确控制返回的数据字段，避免过度获取数据。
- 利用 `where` 子句优化查询条件，减少数据扫描范围。
- 善用 `pagination`（`skip`, `take`）和 `orderBy` 进行分页和排序。

## 5. 错误处理
- 捕获Prisma Client抛出的数据库相关错误（例如：`PrismaClientKnownRequestError`）。
- 将数据库错误转换为友好的业务错误或通用HTTP错误。

## 6. 数据安全 (RLS)
- 在 `schema.prisma` 中定义 `@@auth` 或其他相关规则来指导 Supabase 的 Row Level Security (RLS) 策略的实现。
- 确保后端API的权限控制与数据库RLS策略协同工作，形成纵深防御。

## 7. 迁移管理
- 每次 `schema.prisma` 发生变更时，使用 `npx prisma migrate dev --name <migration_name>` 创建新的迁移文件。
- 仔细审查生成的SQL迁移脚本，确保其正确性。
- 在部署到不同环境时，使用 `npx prisma migrate deploy` 应用迁移。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZENCR-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ZENCR-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
