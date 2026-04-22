## monkeyrestriction

> description: Java 后端开发规范 - Spring Boot/Cloud

---
description: Java 后端开发规范 - Spring Boot/Cloud
globs:
  - "src/main/java/**/*.java"
  - "src/main/resources/**/*"
  - "pom.xml"
  - "application*.yml"
  - "application*.properties"
alwaysApply: true
---

# Spring Boot/Cloud 开发规范

## 项目结构
- 遵循标准的 Maven 项目结构
- 包名按照功能模块划分：`com.deltav.module`
- 每个模块包含：controller, service, repository, model, config 等子包

## 代码规范
- 使用 Lombok 简化代码
- 遵循阿里巴巴 Java 开发手册
- 使用 Swagger/OpenAPI 进行 API 文档管理
- 使用统一的异常处理机制
- Spring bean 注入优先使用构造器注入和 Lombok 注解

## 配置管理
- 使用 YAML 格式的配置文件
- 区分开发、测试、生产环境配置
- 敏感信息使用配置中心管理

## 接口设计
- RESTful API 设计规范
- 统一的响应格式
- 合理的 HTTP 状态码使用
- 接口版本控制

## 数据库
- 使用 JPA/Hibernate 进行 ORM 映射
- 遵循数据库命名规范
- 合理使用索引
- 注意 SQL 注入防护

## 安全
- 使用 Spring Security 进行认证授权
- 敏感数据加密存储
- 防止 XSS、CSRF 等攻击
- 接口访问权限控制

## 性能优化
- 合理使用缓存
- 数据库连接池配置
- 异步处理耗时操作
- 监控系统性能指标

## 测试
- 单元测试覆盖核心业务逻辑
- 集成测试验证接口功能
- 使用 Mock 对象进行测试
- 持续集成环境配置

## 部署
- 使用 Docker 容器化部署
- 配置 CI/CD 流程
- 日志集中管理
- 监控告警机制

## 文档
- 当最终的实现和 `/doc/requirements-solution.md` 中的解决方案不一致时，更新该文档

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DeltaV235) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
