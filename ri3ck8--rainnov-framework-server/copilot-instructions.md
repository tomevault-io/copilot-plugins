## rainnov-framework-server

> 面向 AI 编码助手与新加入开发者的项目索引。**先读本文件，再按需跳转 `docs/` 下的对应文档。**

# AGENTS.md — Rainnov Framework Server

面向 AI 编码助手与新加入开发者的项目索引。**先读本文件，再按需跳转 `docs/` 下的对应文档。**

## 项目一句话

基于 Netty + Protobuf + Spring Boot 4.1 + Java 25 的休闲游戏服务器框架：WebSocket 二进制帧按消息号（msgId）路由到注解标注的 Controller 方法，每用户独立队列 + 虚拟线程保证消息串行。

## 技术栈速览

Java 25（虚拟线程）· Gradle 9.5.1 Wrapper · Spring Boot 4.1.0 · Netty 4.1.115.Final · Protobuf 4.31.1 · Guava RateLimiter · Lombok · JUnit 5 + Mockito + jqwik。

版本口径与 `rainnov-lockstep-server` 对齐，两个项目的 Java / Spring Boot / Protobuf / Gradle 版本须同步升级。

## 常用命令

```bash
./gradlew build          # 完整构建：generateProto → generateMsgId → compileJava → test
./gradlew test           # 仅测试
./gradlew bootRun        # 启动服务器，默认 ws://localhost:8888/ws
./gradlew generateMsgId  # 修改 .proto 后重新生成 MsgId.java
./gradlew clean
```

改完 Java 代码至少跑 `./gradlew compileJava`；改完 `.proto` 必须跑 `./gradlew generateMsgId`。

## 硬性规则（违反会导致启动失败或构建不一致）

1. `src/main/java/com/rainnov/framework/proto/MsgId.java` 是 Gradle 自动生成的，**禁止手动编辑**。
2. Proto 业务消息必须命名为 `C{msgId}_{MessageName}`，请求号为奇数，响应号 = 请求号 + 1。
3. `@MsgMapping` 方法签名固定为 `(GameSession session, XxxReq req)`，且注解里的 msgId 必须与参数类名前缀 `C{msgId}_` 一致，否则启动时 Fail-Fast。
4. 同一 msgId 只能注册一个处理方法，重复注册启动即抛 `IllegalStateException`。
5. 业务逻辑只放在 `com.rainnov.modules.*`，`com.rainnov.framework.*` 保持与业务无关。
6. Spring Bean 统一构造器注入，不用字段 `@Autowired`。

## 文档索引

| 文档 | 内容 |
|---|---|
| [docs/product.md](docs/product.md) | 产品定位、核心能力、目标场景、消息流转概览 |
| [docs/tech-stack.md](docs/tech-stack.md) | 依赖清单、构建链、Gradle 任务、配置项、编码设置 |
| [docs/project-structure.md](docs/project-structure.md) | 包结构、目录职责、proto 源文件布局 |
| [docs/architecture.md](docs/architecture.md) | 框架架构：组件职责、消息分发算法、队列体系、限流、优雅停机、错误码、监控 |
| [docs/protocol.md](docs/protocol.md) | `GameMessage` 协议、msgId 分段规划、`generateMsgId` 生成机制 |
| [docs/conventions.md](docs/conventions.md) | 命名与编码约定（proto / Controller / Service / Model / 错误码） |
| [docs/testing.md](docs/testing.md) | 测试分层、命名约定、属性测试用法 |
| [docs/requirements/framework.md](docs/requirements/framework.md) | 框架 EARS 验收标准（需求 1~13） |
| [docs/requirements/inventory.md](docs/requirements/inventory.md) | 背包模块 EARS 验收标准（需求 1~11） |
| [docs/properties.md](docs/properties.md) | 正确性属性清单（框架 + 背包，含验证的需求编号） |
| [docs/modules/inventory.md](docs/modules/inventory.md) | 背包模块设计与消息、错误码、过期策略 |

### 开发指引（按顺序使用）

| 指引 | 用途 |
|---|---|
| [docs/guides/new-module.md](docs/guides/new-module.md) | 新业务模块端到端脚手架（编排以下各步） |
| [docs/guides/proto-messages.md](docs/guides/proto-messages.md) | 定义 Proto 消息 + 注册消息号范围 |
| [docs/guides/module-models.md](docs/guides/module-models.md) | 数据模型、枚举、配置注册中心、错误码 |
| [docs/guides/module-service.md](docs/guides/module-service.md) | Service 层业务逻辑与 Result record |
| [docs/guides/module-controller.md](docs/guides/module-controller.md) | Controller 层消息处理方法 |
| [docs/guides/handler-strategy.md](docs/guides/handler-strategy.md) | 策略模式 Handler + Registry（可选） |

## 其他入口

- `README.md` — 面向使用者的快速上手。
- 所有设计、约定、需求与开发指引统一放在 `docs/`，本文件是唯一索引入口；不再维护 `.kiro/` 下的 steering / skills / specs 副本。

---
> Source: [ri3ck8/rainnov-framework-server](https://github.com/ri3ck8/rainnov-framework-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
