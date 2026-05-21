## all

> description: CodeSpirit 全局开发规范概览 - 核心原则和规范索引

﻿---
description: CodeSpirit 全局开发规范概览 - 核心原则和规范索引
globs: 
alwaysApply: true
---

# 通用要求

- 严格遵守项目结构规范，保持代码组织一致性
- EF 迁移按数据库目录创建（SqlServer/MySql）
- 分布式程序优先考虑：高可用、容错、降级、最终一致性
- 所有面向用户的文本必须支持多语言（中文/英文）
- 异步编程：所有 I/O 操作使用 `async/await`，禁止 `Task.Result` 和 `Task.Wait()`

> 💡 **详细规范**: 本文件是全局概览，详细规范请参考专项文档

# 技术栈

- **框架**: .NET 10 + Aspire 13.0
- **数据库**: MySQL 8.0 / SQL Server 2022 + GreptimeDB（审计）
- **缓存/消息**: Redis + RabbitMQ
- **前端**: React + AMIS (AntD 主题)
- **AI**: OpenAI、通义千问、DeepSeek

# 核心规范

## 统一启动框架
- Program.cs 仅需 2 行代码 + 配置类继承 `BaseApiConfiguration`
- 📖 [详细规范](mdc:.cursor/rules/startup-framework.mdc)

## 依赖注入
- 自动注册：`IScopedDependency` / `ITransientDependency` / `ISingletonDependency`
- 📖 [详细规范](mdc:.cursor/rules/dependency-injection.mdc)

## 命名约定
- 实体：`User`、DTO：`CreateUserDto`、服务：`UserService`、控制器：`UsersController`
- 📖 [详细规范](mdc:.cursor/rules/naming-conventions.mdc)

## API 设计
- RESTful 标准 + 统一响应格式 `ApiResponse<T>` + 操作特性标记
- 📖 [详细规范](mdc:.cursor/rules/api-design.mdc)

## 多语言国际化
- 支持中英文 + 资源文件 + DTO 验证特性多语言
- 📖 [详细规范](mdc:.cursor/rules/i18n.mdc)

## AI 功能开发
- AI 表单填充（`[AiFormFill]`）+ AI 长任务（`aiForm`）+ LLM 集成
- 📖 [详细规范](mdc:.cursor/rules/ai-development.mdc)

## 性能优化
- 异步编程 + 多级缓存（L1+L2）+ EF Core 查询优化
- 📖 [详细规范](mdc:.cursor/rules/performance.mdc)

## 安全规范
- 权限控制（`[Authorize]` + `[RequirePermission]`）+ 审计追踪 + 数据加密
- 📖 [详细规范](mdc:.cursor/rules/security.mdc)

## 包管理规范
- 集中式包管理：所有包版本在 `Directory.Packages.props` 中统一管理
- 项目文件中只引用包名，不指定版本
- 避免冗余引用：通过传递依赖可用的包不需要显式引用
- 📖 [详细规范](mdc:.cursor/rules/package-management.mdc)

# 数据访问

- **多数据库支持**: 使用数据库特定 DbContext（`SqlServer{Service}DbContext` / `MySql{Service}DbContext`）
- **Code First 迁移**: 按数据库创建独立迁移（SqlServer/MySql），**必须使用数据库特定 DbContext**
- **雪花 ID 配置**: 使用 `IIdGenerator` 的实体必须配置 `ValueGeneratedNever()`
- **实体配置**: `IEntityTypeConfiguration<T>`
- **只读查询**: `AsNoTracking()`，避免 N+1 用 `Include` + `AsSplitQuery()`
- **多租户**: 实体实现 `IMultiTenant`，自动应用租户过滤器
- 📖 [详细规范](mdc:.cursor/rules/database.mdc)

# 代码质量

- 一个 .cs 文件一个顶级类型
- XML 文档注释：`<summary>`, `<param>`, `<returns>`, `<exception>`
- 时间使用 UTC 格式

# 组件使用

- **AMIS**: `antd` 主题，CSS 类用 `antd-` 前缀，特性驱动
- **LLM**: 使用 `ILLMClientFactory`，提示词指定 JSON 输出

# 调试运行

- **启动**: `CodeSpirit.AppHost` (Aspire 协调) → `aspire run` 或 F5
- **Dashboard**: Aspire 管理面板
- **健康检查**: `/health`

# 项目结构

```
Src/
├── ApiServices/        # 14个API服务（Identity, Exam, Survey, AI等）
├── Components/         # 多个核心组件（Amis, LLM, Caching等）
├── CodeSpirit.AppHost/ # Aspire应用宿主
├── CodeSpirit.Core/    # 核心框架
└── CodeSpirit.Web/     # Web前端
```

📖 [完整项目结构](mdc:.cursor/rules/project-structure.mdc)

# 规范文档索引

## 通用规范
- [C#通用规范](mdc:.cursor/rules/cs.mdc) - XML注释、时间格式、序列化
- [命名约定](mdc:.cursor/rules/naming-conventions.mdc) - 实体、DTO、服务、控制器命名

## 按文件类型
- [DTO规范](mdc:.cursor/rules/dto.mdc) - DTO特性、验证、映射
- [控制器规范](mdc:.cursor/rules/controller.mdc) - API控制器、操作特性
- [服务类规范](mdc:.cursor/rules/service.mdc) - 服务接口、实现、生命周期
- [枚举规范](mdc:.cursor/rules/enum.mdc) - 枚举定义、多语言支持

## 专项规范
- [API设计](mdc:.cursor/rules/api-design.mdc) - RESTful、路由、响应格式
- [依赖注入](mdc:.cursor/rules/dependency-injection.mdc) - Scrutor自动注册
- [启动框架](mdc:.cursor/rules/startup-framework.mdc) - Program.cs配置
- [数据库迁移](mdc:.cursor/rules/database.mdc) - 多数据库、DbContext、迁移命令
- [多语言](mdc:.cursor/rules/i18n.mdc) - 资源文件、本地化
- [AI开发](mdc:.cursor/rules/ai-development.mdc) - AI表单、长任务、LLM
- [性能优化](mdc:.cursor/rules/performance.mdc) - 异步、缓存、查询
- [安全规范](mdc:.cursor/rules/security.mdc) - 权限、审计、加密
- [测试规范](mdc:.cursor/rules/testing.mdc) - 单元测试、集成测试、Mock使用

## 项目结构
- [项目结构](mdc:.cursor/rules/project-structure.mdc) - 完整目录树

## 包管理
- [包管理规范](mdc:.cursor/rules/package-management.mdc) - 集中式包管理、版本统一、传递依赖处理

> 💡 专项规范会根据文件类型自动应用

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
