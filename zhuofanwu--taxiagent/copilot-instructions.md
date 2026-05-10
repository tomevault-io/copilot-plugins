## taxiagent

> 本文件为 Codex (OpenAI Codex / OpenCode) 在处理本仓库代码时提供相关指导。

# AGENTS.md

本文件为 Codex (OpenAI Codex / OpenCode) 在处理本仓库代码时提供相关指导。

## 项目概述

TaxiAgent 是一个基于 Spring Boot 3 的 AI 驱动的出租车预订和客户服务平台，包含 4 个专用的 AI 代理（DailyAgent、OrderAgent、SupportAgent、FallbackAgent）。

## 构建与运行命令

```bash
mvn compile                    # 编译项目
mvn compile -DskipTests        # 编译并跳过测试
mvn clean package              # 构建 JAR 文件
mvn test                       # 运行单元测试
mvn spring-boot:run            # 直接运行应用程序
```

## 核心技术栈

- **语言**: Java 21
- **框架**: Spring Boot 3.5.9 与 Spring AI Alibaba Agent 1.1.0.0-RC2
- **数据库**: MySQL（用户、订单、工单），MongoDB（订单路线、对话记录）
- **搜索与缓存**: Elasticsearch（聊天全文检索），Redis（缓存）
- **ORM**: MyBatis-Plus 3.5.9
- **大模型供应商**: DashScope（向量化/Embedding），DeepSeek（兼容 OpenAI API）

## 架构

### AI 代理层 (`agentbase/agents/`)
- 4 个专用代理负责处理不同领域的任务
- 记忆（Memory）系统跨越 Redis、MongoDB、MySQL 和 Elasticsearch 进行存储
- 用于函数调用的工具回调（Tool callbacks）位于 `agentbase/tool/` 目录

### API 层 (`controller/`)
- 采用 控制器 (Controllers) -> 服务 (Services) -> 映射器 (Mappers) 架构模式
- 基于 Token 的身份验证，支持邮箱一次性密码 (OTP)
- API 文档位于 `src/main/resources/api/`

### 数据层
- MySQL: 关系型数据（用户、订单、工单、聊天日志）
- MongoDB: 订单路线和对话历史记录
- Elasticsearch: 基于 N-Gram 的聊天全文检索

## 开发注意事项

- 在 `agentbase.amap` POJO 类中，对于返回类型为 String 的字段，请使用 `StringOrEmptyArrayToNullDeserializer` 进行反序列化
- 创建新工具时，请遵循现有的函数模式
- 在服务层（services）中使用 MyBatis-Plus 的 LambdaQueryMapper
- 修改控制器（controllers）时，请同步更新 `resources/api/` 中的 API 文档
- 应用补丁（patch）时，请使用相对路径，例如 `src/main/java/com/fancy/taxiagent/controller/AuthController.java`

---
> Source: [ZhuofanWu/TaxiAgent](https://github.com/ZhuofanWu/TaxiAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
