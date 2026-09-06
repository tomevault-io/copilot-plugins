## spring-ai-code-demo

> 欢迎来到 **spring-ai-code** 仓库。本文件为整个仓库提供统一的协作规范和执行指引，除非在子目录中出现更具体的 `AGENTS.md` 文件，本指南适用于仓库内的全部文件。

# AGENTS

欢迎来到 **spring-ai-code** 仓库。本文件为整个仓库提供统一的协作规范和执行指引，除非在子目录中出现更具体的 `AGENTS.md` 文件，本指南适用于仓库内的全部文件。

## 🧭 项目概览
- 本项目是一个基于 **Spring Boot 3.5.5** 与 **Spring AI 1.1.0-SNAPSHOT** 的多模块 MCP 服务器，核心模块包含 `spring-ai-code-start`、`spring-ai-code-common`、`spring-ai-code-model`、`spring-ai-code-mcp`、`spring-ai-code-mcp-ailpha`、`spring-ai-code-web` 与 `spring-ai-code-ai`。请保持模块职责清晰，避免跨模块的循环依赖。
- 统一使用 Java 17。若需引入新依赖，请确认其与 Java 17 兼容且能在多模块 Maven 架构下正常工作。

## ✍️ 代码与文档规范
- **命名与风格**：保持驼峰命名，公共 API 与 DTO 必须使用强类型，严禁以原始 `Map` 代替领域对象。必要时补充 Lombok 注解，但不得牺牲可读性。
- **注释与日志**：新增或修改的业务代码必须配备中文注释与结构化日志（SLF4J），并补齐关键方法的 JavaDoc。
- **响应模型**：面向外部的接口请继续沿用 `ApiResponse` / `ApiErrorResponse` 与 `DataResponse<T>` 统一响应封装，确保时间戳与 success 字段正确设置。
- **异常处理**：优先使用现有的统一异常体系，确保控制层不会抛出未检查异常。
- **设计原则**：遵循分层架构与既有设计模式（策略、单例、门面等）。新增策略或组件时，请在相应模块内补齐注册逻辑与单元测试。
- **文档同步**：若变更涉及接口、配置或使用方式，请同步更新 `docs/` 目录下的相关指南或 README，并确保中文描述准确。

## ✅ 测试与校验
在提交任何修改前必须完成以下检查：
1. `mvn clean compile` - 验证所有模块可以成功编译。
2. `mvn test` - 运行全量单元测试。
3. 如变更涉及集成流程或 MCP 协议，请额外执行 `mvn verify` 或针对性地运行相关测试类（如 `mvn test -Dtest=McpIntegrationTest`）。
4. 若仅修改单一模块且编译耗时过长，可先通过 `mvn clean compile -pl <module>` / `mvn test -pl <module>` 做增量验证，最终仍需通过全量命令。

请在最终提交前确保 `mvn` 命令全部通过，并将执行结果附在工作报告中。

## 🗂️ 文件组织与资源
- 公共常量、工具类放在 `spring-ai-code-common`；数据模型放在 `spring-ai-code-model`；MCP 相关逻辑放在 `spring-ai-code-mcp`；业务扩展放在 `spring-ai-code-mcp-ailpha`。
- 若新增配置，需在对应模块的 `resources` 下补充示例，并在 `docs/guides/快速开始指南.md` 或相关文档中记录使用说明。
- Web 或 AI 前端改动须同步准备静态资源结构说明，必要时补充截图展示效果。

## 🧪 质量门槛
- 新增功能必须具备相应的单元或集成测试，测试需覆盖主要分支逻辑并确保可重复执行。
- 日志请避免输出敏感数据（如 API Key、密码等），若必须记录请进行脱敏处理。
- 依赖版本更新需同时评估对 Docker 与部署脚本的影响，必要时更新 `docker/` 或 `build-optimized.sh`。

## 📝 提交与评审
- 遵循小步提交原则，确保每次提交可独立通过构建与测试。
- PR 描述应包含变更摘要、影响范围、测试结果以及是否需要额外部署步骤。
- 若遇到需要新增子目录说明的情况，请在子目录创建新的 `AGENTS.md`，并在其中记录额外约束。

祝编码顺利！

---
> Source: [wwj-git-rgb/spring-ai-code-demo](https://github.com/wwj-git-rgb/spring-ai-code-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
