## water

> - `water-server/`：Spring Boot 后端，主代码在 `src/main/java/com/water/server`，配置和建表脚本在 `src/main/resources`，测试在 `src/test/java`。

# Repository Guidelines

## 项目结构与模块划分
仓库由两个应用组成：

- `water-server/`：Spring Boot 后端，主代码在 `src/main/java/com/water/server`，配置和建表脚本在 `src/main/resources`，测试在 `src/test/java`。
- `water-web/`：Vue 3 + Vite 前端，入口和页面逻辑在 `src/`，组件集中在 `src/components`。
- 根目录下的 `water.db`、`water.sql`、`water.csv` 用于本地开发和数据初始化，运行日志输出到 `runtime-logs/`。
- `water-server/target/` 和 `water-web/dist/` 属于构建产物，不应作为手工维护内容。

## 构建、测试与开发命令
- `cd water-server && mvn spring-boot:run`：启动后端，默认地址 `http://localhost:8080`。
- `cd water-server && mvn test`：运行后端 JUnit 测试。
- `cd water-server && mvn package`：构建后端 JAR，输出到 `target/`。
- `cd water-web && npm run dev`：启动前端开发服务器，默认地址 `http://localhost:5173`。
- `cd water-web && npm run build`：构建前端生产包，输出到 `dist/`。
- `cd water-web && npm run preview`：本地预览构建结果。

前端会将 `/api/*` 代理到后端，因此本地开发时应先启动 `water-server`，再启动 `water-web`。

## 代码风格与命名约定
- Java 使用 4 空格缩进，遵循 Spring Boot 常规写法；类名使用 `UpperCamelCase`，方法和字段使用 `lowerCamelCase`。
- Vue/JavaScript 使用 2 空格缩进；组件文件采用 `PascalCase`，如 `SnapshotEditor.vue`；变量和函数使用 `camelCase`。
- 后端按功能聚合在 `com.water.server.snapshot` 包下，控制器、服务、DTO 保持同一业务域内聚。
- 类型命名应直接表达用途，例如 `AssetSnapshotUpsertRequest`、`AssetAccountCommandService`。

## 测试要求
- 后端测试基于 `spring-boot-starter-test` 和 JUnit 5，位置在 `water-server/src/test/java`。
- 测试类统一使用 `*Tests` 后缀，测试方法使用清晰的行为描述，例如 `createsUpdatesAndDeletesSnapshot`。
- 当前仓库没有独立的前端测试框架；提交前至少执行一次 `npm run build`，确认打包和基础集成无误。

## 提交与 Pull Request 规范
- 当前提交历史中既有简短提交，也有 `style: ...` 这类更清晰的写法。后续建议统一使用 `<type>: 简要说明`。
- 每次提交尽量只处理一个关注点，例如接口改动、样式调整、数据脚本更新分开提交。
- PR 需要说明修改范围、影响模块（`water-server` 或 `water-web`）、手动验证步骤；涉及界面变更时附上截图。

## 配置说明
- SQLite 数据源配置位于 `water-server/src/main/resources/application.yml`。
- 不要新增未说明的机器本地绝对路径；如果必须使用，应在 PR 描述中写明原因和替代方式。

---
> Source: [wangxpcool/water](https://github.com/wangxpcool/water) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
