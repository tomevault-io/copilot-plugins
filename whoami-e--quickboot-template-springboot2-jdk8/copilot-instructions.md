## quickboot-template-springboot2-jdk8

> QuickBoot — Spring Boot 2.7.18 + JDK 8 多模块 DDD 分层 Maven 脚手架。基础包名 `com.quickboot`，端口 `8080`，默认 `dev` 环境无需数据库/Redis 即可启动。

# AGENTS.md

QuickBoot — Spring Boot 2.7.18 + JDK 8 多模块 DDD 分层 Maven 脚手架。基础包名 `com.quickboot`，端口 `8080`，默认 `dev` 环境无需数据库/Redis 即可启动。

## 验证命令

```bash
mvn -B test
```

CI 在 push(master/dev) 和 PR(master) 时运行此命令；auto-merge 依赖 CI 成功后自动合并。

## 模块与分层规则

依赖方向自上而下，禁止反向依赖：

```
quickboot-web          → Controller / 全局异常处理 / 过滤器 / 启动类
quickboot-application  → 应用服务层（编排业务逻辑）
quickboot-infrastructure → 数据访问 / MyBatis-Plus / Redis / Sa-Token 配置
quickboot-domain       → 领域模型（纯 POJO，不含 Spring 注解）
quickboot-common       → 统一响应 / 错误码 / 异常 / 工具类
```

- Controller 只做请求接收与响应返回，返回值统一为 `ApiResponse<T>`
- Domain 模型不含 Spring 注解
- 工具类使用 `final` + 私有构造器，禁止实例化
- 统一使用构造器注入，禁止字段注入
- 新增可选组件遵循 optional 依赖 + `@ConditionalOnClass` + `@ConditionalOnProperty` 三重保护

详细命名规范、Lombok 使用和开发流程见 [README.md](README.md) 开发约定章节和 [SKILL.md](SKILL.md)。

## 高风险操作边界

- **数据库变更**：无 Flyway/Liquibase 迁移工具；DB 模式通过 `application-*.yml` 中 `mybatis-plus.enabled` 开关控制，未启用时 UserService 回退内存模式
- **auto-merge**：CI 成功后自动合并 PR 到 master，具有 `pull-requests: write` + `contents: write` 权限
- **可插拔组件**：Sa-Token 和 Redis 为 optional 依赖，移除需同步清理对应 `@ConditionalOnClass` 配置
- **外部写入**：Docker 镜像构建和多环境部署配置在 `Dockerfile` 和 `docker-compose.yml` 中

---
> Source: [Whoami-e/quickboot-template-springboot2-jdk8](https://github.com/Whoami-e/quickboot-template-springboot2-jdk8) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
