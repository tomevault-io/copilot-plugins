## ai-agent-learn

> - 多模块 Maven 项目，主模块为：`langchain4j-ai`

# AGENTS.MD - AI 代理学习仓库指南
## 对话
- 必须用中文对话
## 项目概述
- 多模块 Maven 项目，主模块为：`langchain4j-ai`
- 基于 Spring Boot 4.1.0-M2、Java 25 和 Spring AI 2.0.0-M3 构建
- 集成了 LangChain4j 1.11.0，支持 Redis、Milvus 和字节跳动火山引擎模型

## 必需的服务设置
```
# 运行测试或应用程序之前，请先启动以下外部服务：
# 1. 通过 Docker Compose 启动 Redis (如 README 中所示)
# 2. Milvus 向量数据库：etcd、minio 和独立组件 (如 README 中所示)
# 端口 6379: Redis, 端口 19530 & 9091: Milvus
```

## 开发命令
- 构建: `mvn clean install`
- 运行: `mvn spring-boot:run -pl langchain4j-ai`
- 测试: `mvn test -pl langchain4j-ai`

## 架构说明
- AI 内存存储使用 Redis 配置
- 矢量嵌入搜索通过 Milvus 连接执行
- 包含自定义 EmailTool 和嵌入加载任务
- 配置于 LangChain4jConfig 和 application.properties 中

## 测试前提条件
- 运行测试前必须启动外部服务 (Redis、Milvus)
- 集成测试需要使用 README.md 中设置的 Docker 容器

## 核心模块
- `com.civism.ai.controller`: 聊天和嵌入搜索控制器
- `com.civism.ai.memory.redis`: 基于 Redis 的内存存储
- `com.civism.ai.config`: AI 模型和代理配置
- `com.civism.ai.task`: 后台嵌入加载任务

---
> Source: [civism-bit/ai-agent-learn](https://github.com/civism-bit/ai-agent-learn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
