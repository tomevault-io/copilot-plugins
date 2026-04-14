## thunder-note-server

> `thunder-note-server/` 是当前真实后端实现：Spring Boot 3 + Java 21 + MyBatis-Plus + PostgreSQL + Redis + Flyway + JWT + MinIO。整体按领域分包，但实现范围仍小于旧设计文档描述的目标架构。

# 后端知识库

## 概览
`thunder-note-server/` 是当前真实后端实现：Spring Boot 3 + Java 21 + MyBatis-Plus + PostgreSQL + Redis + Flyway + JWT + MinIO。整体按领域分包，但实现范围仍小于旧设计文档描述的目标架构。

## 高频入口
| 任务 | 位置 | 说明 |
|------|------|------|
| 应用入口 | `src/main/java/com/flashnote/FlashNoteApplication.java` | `@SpringBootApplication` + `@MapperScan` |
| 认证接口 | `src/main/java/com/flashnote/auth/controller/AuthController.java` | 当前真实路径 `/api/auth/...` |
| 安全链路 | `src/main/java/com/flashnote/common/config/SecurityConfig.java` | JWT + 无状态认证 |
| 统一响应 | `src/main/java/com/flashnote/common/response/ApiResponse.java` | 统一响应包装 |
| 数据库事实 | `src/main/resources/db/migration/V1__initial_schema.sql` | 真实 schema 来源 |
| 运行配置 | `src/main/resources/application.yml` | PostgreSQL、Redis、MinIO、JWT |

## 本模块特有约定
- 其他通用规则遵循根目录 `AGENTS.md`。
- 遵循 controller → service → mapper/entity 分层。
- 默认返回 `ApiResponse`，不要绕开统一响应与全局异常处理。
- 数据库事实以 Flyway 迁移脚本为准，不以旧数据库设计文档为准。
- 改接口前，除了核对控制器与服务，也要顺手核对 Android `data/remote/api/` 是否继续漂移。

## 当前实现与旧文档差异
- 当前接口走 `/api/...`，不是 `/api/v1/...`。
- 登录使用 `username` 字段，不是文档长期描述的邮箱登录。
- 当前 schema 与同步实现都比旧设计文档更基础。

## 反模式
- 不要直接按照旧文档里的路径或请求结构实现，必须先核对 controller。
- 不要只根据 `数据库设计.md` 推断 schema，必须同时核对 Flyway 与实体。
- 不要假设已有测试覆盖；要靠实际构建和运行验证。

## 备注
- 需重点关注的模块：`auth/`、`sync/`、`file/`、`common/`。
- 如果文档、控制器、Android API 声明三者不一致，默认先以控制器与运行配置为准。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/warmsheep) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
