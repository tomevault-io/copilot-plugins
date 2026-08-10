## database

> 数据库、ORM 模型、Alembic 迁移编写规范


# 数据库 & ORM 规范

> 本规则是**可执行约定**（怎么写）。约定背后的设计理由（为何不用 ForeignKey、软删除架构选型、JSONB 索引策略、ORM 为何是 schema 真相源）见 `docs/02-架构/核心接口定义.md` §6.2，勿在此重复展开。

## SQLAlchemy 模型

- 使用 SQLAlchemy 2.0 `Mapped` + `mapped_column` 声明式
- **所有表主键**：`PG_UUID(as_uuid=False)`，应用层 `str(uuid4())` 生成
- **默认值必须用 `server_default`**（seed 脚本和裸 SQL 不触发 Python 级 default）
- 枚举字段用 `CheckConstraint` 固化合法值
- **不使用 ForeignKey**（应用层维护引用完整性），外键字段加 `index=True`
- JSONB 字段同时设 `default=dict/list` 和 `server_default=text("'{}'::jsonb")`
- JSONB 值与 UUID 列比较时用 `cast(..., PG_UUID(as_uuid=False))`
- 软删除：设置 `deleted_at` 而非物理删除
- **ORM 是 schema 单一真相源**：`alembic check` 必须零漂移

### 标准字段

| 字段 | 类型 | 要求 |
|---|---|---|
| `id` | `PG_UUID(as_uuid=False)` | 主键 |
| `created_at` | `DateTime(timezone=True)` | `server_default=func.now()` |
| `updated_at` | `DateTime(timezone=True)` | `server_default=func.now(), onupdate=func.now()` |
| `deleted_at` | `DateTime(timezone=True), nullable` | 软删除标记 |
| `status` | `String(20)` | 配 CheckConstraint |

## Repository 层

- 只做数据访问，不含业务判断
- 查询用 `select()` 构建式，不用 raw SQL
- 分页返回 `tuple[list[Model], int]`（数据 + 总数）
- 默认按 `created_at.desc()` 排序
- `commit()` 和 `refresh()` 由 repository 负责

## Alembic 迁移

- schema 与数据严格分离：结构走迁移，初始数据走 seed 脚本
- `upgrade()` 和 `downgrade()` 必须成对实现
- 复杂数据迁移用 `op.execute()`

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
