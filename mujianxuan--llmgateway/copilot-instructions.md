## llmgateway

> │   ├── java/org/ai/llmGateway/  # 主源代码

# AGENTS 贡献者指南

## 项目结构和模块组织

```
src/
├── main/
│   ├── java/org/ai/llmGateway/  # 主源代码
│   └── resources/               # 配置文件 (application.yaml)
├── test/                        # 测试代码
.docs/                           # 项目文档
```

主要技术栈：Spring Boot 3.5.8 + Java 21，集成Anthropic和OpenAI API，使用JPA + SQLite持久化。

## 构建、测试和开发命令

```bash
# 后端命令
mvn compile              # 编译项目
mvn test                 # 运行测试
mvn package              # 打包应用
mvn spring-boot:run      # 启动应用
java -jar target/llmGateway-0.0.1-SNAPSHOT.jar  # 运行打包后的JAR

# 前端命令（在 frontend/ 目录下执行）
npm run dev              # 开发模式启动（Vite）
npm run build            # 构建生产版本
npm run lint             # 执行 ESLint 检查
```

## 代码风格和命名约定

- **Java版本**: 21
- **包命名**: `org.ai.llmGateway.*`
- **注解处理**: 使用Lombok简化代码
- **测试框架**: JUnit 5 + TestContainers（集成测试）+ H2（内存数据库）
- **缩进**: 4空格，遵循IntelliJ IDEA默认格式
- **测试类命名**: `*Test.java`，位于 `src/test/java` 目录
- **测试覆盖率**: 确保新增功能包含相应的单元测试和集成测试

## 安全实践

- **API密钥加密**: 提供商API密钥使用AES-256-GCM加密存储
- **密钥哈希**: 网关API密钥使用SHA256哈希存储
- **输入验证**: 所有API请求参数必须通过Spring Validation校验
- **SQL注入防护**: 使用JPA参数化查询，禁止拼接SQL
- **HTTPS**: 生产环境强制使用HTTPS传输

## 附加说明

- 使用AOP实现横切关注点（请求日志、认证）
- 开发时使用Spring Boot DevTools实现热重载
- 数据库使用SQLite（生产）和H2（测试）
- 所有外部依赖版本在 `pom.xml` 中统一管理
- 流式响应采用SSE（Server-Sent Events）协议
- 健康检查间隔30秒，连续3次失败标记为unhealthy

## 测试规则

- 先怀疑是否已经启动了服务，不要直接启动前后端服务；

---
> Source: [MuJianxuan/LlmGateway](https://github.com/MuJianxuan/LlmGateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
