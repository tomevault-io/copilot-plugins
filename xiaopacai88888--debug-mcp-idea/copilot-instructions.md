## debug-mcp-idea

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目简介

IntelliJ IDEA 插件，内嵌一个 MCP（Model Context Protocol）server，把 IDE 的调试能力
（断点 / 步进 / 读变量 / 求值等）通过 MCP 协议暴露给 AI（对接 Claude Code / Desktop）。

进度见 `PHASES.md`（五阶段路线图）：阶段一~四（脚手架链路、断点管理、状态感知+执行控制、数据检查）
均已完成并经 `runIde` 联调；阶段五打磨中，5.1 多窗口/多会话、5.2 健壮性、5.3 易用性配置均已完成，
5.4 MCP 专属 Skill（见 `skills/intellij-debugging/`）、5.5 使用说明（根目录 `INSTALL.md` + `USAGE.md`）、
5.6 打包安装（`buildPlugin` 产 zip + 插件图标 + `until-build=261.*`）构建侧均已完成，余 5.6「IDEA 从磁盘安装」人类在环验证。当前已实现的 MCP 工具：
`ping` / `list_projects` / `set_breakpoint` / `remove_breakpoint` / `list_breakpoints` / `toggle_breakpoint` /
`list_sessions` / `get_debug_status` / `get_stack_trace` / `resume` / `pause` /
`step_over` / `step_into` / `step_out` / `read_variables` / `read_variable` / `evaluate_expression` /
`list_consoles` / `read_console`。

## 常用命令

在 Windows + PowerShell 环境下使用 wrapper：

```powershell
.\gradlew.bat build          # 编译 + 构建插件
.\gradlew.bat compileKotlin  # 只编译 Kotlin（最快的语法/依赖校验）
.\gradlew.bat runIde         # 启动带本插件的沙箱 IDEA，验证 server 是否监听端口
.\gradlew.bat runIdeSecond   # 起第二个独立 sandbox 实例（验端口探测：8765 被占→落 8766）
.\gradlew.bat buildPlugin    # 打包可分发的插件 zip（build/distributions/debug-mcp-<ver>.zip）
.\scripts\make-release.ps1 -Build  # 重建 release/ 发布包：buildPlugin + 聚合 Skill/USAGE + 打 bundle zip
```

**没有测试代码**，不要假设存在 test task 或测试约定。验证手段是 `runIde` 起沙箱 IDEA 后做联调。
> 普通 `runIde` 与 `runIdeSecond` 须用不同 sandbox 才能真正并发两个进程（同 sandbox 会触发 IntelliJ 单实例转交）。

> 改动 MCP server / 工具后，务必提醒用户在 Claude Code / Desktop 端**重连**以刷新工具列表。

## 关键构建约束（改动 build 配置前必读）

版本是一条**连锁约束链**，不能单独升降其中一环：
MCP SDK 0.12.0 → 要求 Kotlin 2.3 → 要求 IntelliJ 2026.1 平台 → 要求高版本 JDK。

- **本地平台依赖**：`build.gradle.kts` 用 `local("D:/WindowsApp/JetBrains/IntelliJ IDEA 2026.1")`
  指向本地已装的 IDEA，避免下载约 1.5GB 平台。换机器需改这个硬编码路径。
- **编译 toolchain = IDEA 自带 JBR 25**：`gradle.properties` 里
  `org.gradle.java.installations.paths` 指向 IDEA 的 `jbr`，且禁止自动下载 JDK。
  必须用 JBR 25 才能读取 2026.1 平台的高版本字节码；**系统 JDK 17 无法编译本插件**。
- **产出 target = JVM 21**：Kotlin 暂不支持 target 25，21 更稳且 IDEA 可加载。
  （Gradle daemon 仍跑在系统 JDK 上，编译器跑在 JBR 25 上 —— 两者不同是正常的。）
- **Gradle 9.1.0**：JDK 25 需要 Gradle 9.1.0+，8.x 在 JDK 25 上会崩。
- **必须 exclude kotlinx-coroutines**：MCP SDK 和 Ktor 都传递依赖 coroutines，
  会与 IntelliJ 平台自带的协程冲突。两个 `implementation` 块都 `exclude` 掉 coroutines-core，
  统一用平台版本，否则运行时出现重复/冲突的协程类。
- **MCP API 包路径**：0.12.0 的类型在 `io.modelcontextprotocol.kotlin.sdk.types.*`。

## 架构

整个 IDE 进程只跑**一个** MCP server，默认监听 `http://127.0.0.1:8765/mcp`（HOST/PATH 见
`McpServerService` 的 companion object；**首选端口来自 `McpSettings` 持久化配置**，缺省 `DEFAULT_PORT=8765`）。
首选端口被占（多 IDE 实例）时在 `[首选端口, 首选端口+20)` 内探测落到其它端口，实际端口见 `boundPort`、
运行态见 `runState`（STOPPED/RUNNING/FAILED）、实际端点见 `currentEndpoint()`。传输层是 Ktor CIO + SSE，
`mcp { }` helper 挂载 server。

### 启动与生命周期

- **`McpServerService.kt`** — 应用级（`Service.Level.APP`）单例，核心。负责 server 生命周期
  （`startIfNeeded` / `stop` / `dispose`）与**全部工具注册**。`startIfNeeded` 用 `AtomicBoolean`
  做**幂等**，重复调用只真正启动一次。
- **`McpAppLifecycleListener.kt`** — 主触发：IDE 主界面创建（`appFrameCreated`）即拉起 server。
- **`McpStartupActivity.kt`** — 兜底触发：项目打开后（`ProjectActivity`）再调一次 `startIfNeeded`。

两个启动入口都在 `plugin.xml` 注册，互为兜底，靠 `startIfNeeded` 的幂等保证只起一个 server。
`startIfNeeded` 开头尊重 `McpSettings.enabled`（禁用则直接 return、不绑定）；`restart()` = `stop()`→`startIfNeeded()`，供设置面板按新配置显式重启。

### 源文件分工（按职责拆分，工具注册集中在 service，逻辑落在 *Support）

- **`McpServerService.kt`** — `buildMcpServer()` 里用 `register*Tools(server)` 分组注册全部工具
  （project / breakpoint / session / debug / inspection / console），工具体只做参数解析 + 调用顶层 support 函数。
- **`McpToolSupport.kt`** — 跨工具公共件：`ToolException`（可预期错误，转 `isError` 结果而非中断连接）、
  `toolResult { }` 包裹器、参数读取扩展（`requireString`/`optInt`…）、`resolveSingleProject()`。
- **`DebugSessionSupport.kt`** — 阶段三支撑：会话解析、状态守卫
  （`requirePausedSession`/`requireRunningSession`）、异步→协程桥（`awaitStopAfter`/`collectFrames`）、
  EDT 桥 `onEdt { }`、状态/落点/栈帧 JSON 构造、`controlOutcomeJson`（控制动作的统一「触发→等落点→超时按 running 返回」）。
- **`DataInspectionSupport.kt`** — 阶段四支撑：帧解析、三座异步回调→协程桥（子节点/值呈现/求值）、
  presentation 富文本→文本翻译、递归展开（深度 `MAX_DEPTH_CAP` + 广度 `BREADTH_LIMIT` 双上限）、变量/求值 JSON。
- **`ConsoleSupport.kt`** — 控制台读取支撑（阶段五新增）：跨工程枚举运行/调试 tab（`RunContentManager.allDescriptors`）、
  目标 tab 解析（显式 `target` 走注册表 / 缺省取 `selectedContent`）、包装型 console 解包
  （`ConsoleViewWrapperBase`/`DuplexConsoleView` → `ConsoleViewImpl`，解不出降级 `CONSOLE_UNSUPPORTED`）、
  EDT 上 `flushDeferredText`+`getText` 取尾部 N 行、`list_consoles`/`read_console` 的 JSON 构造。
- **`McpSettings.kt`** — 易用性配置（阶段 5.3 新增）：应用级 `PersistentStateComponent`（`@Storage("debugMcp.xml")`），
  持久化首选端口 `port`（缺省 `DEFAULT_PORT`）与启用开关 `enabled`（缺省 true）；缺省值保证未配置用户向后兼容。
- **`McpConfigurable.kt`** — 设置面板（阶段 5.3 新增）：应用级 `applicationConfigurable`（设置 / 工具 下），
  展示运行态 + 实际端点、改端口、切换启用开关、「应用并重启 server」（后台线程调 `restart()` 避免阻塞 EDT）、
  展示 + 一键复制 MCP 配置片段（只读、不碰外部 Claude 配置文件）；`apply` 校验端口 1024–65535、非法抛 `ConfigurationException`。
- **`McpNotifications.kt`** — 启动期用户可见通知（阶段 5.3 新增）：`NotificationGroup id="DebugMcp"`，
  落到非首选端口弹 info（附实际端点）、启动失败弹 error（附原因）；只在幂等保护内的实际绑定 / 失败路径上调用，不刷屏。
- **`BreakpointRegistry.kt`** / **`SessionRegistry.kt`** / **`ConsoleRegistry.kt`** — 字符串 id
  （`bp-1` / `s1` / `c1`）↔ 平台对象的双向映射。反向用 `IdentityHashMap`（不依赖 `equals`）；惰性接管
  （含用户在 IDE 手动创建的断点/会话）、失效即剔除。`SessionRegistry`/`ConsoleRegistry` 是 `object`
  （与 server 单例对应），`BreakpointRegistry` 是 service 持有的实例。

### 两条硬性约定（新增/修改工具必守）

1. **线程模型**：MCP 工具体跑在 Ktor IO 协程上，**不能**阻塞，也**不能**直接碰平台对象。
   - 读模型用 `readAction { }`，写断点等用 `edtWriteAction { }`，触发调试动作（resume/pause/step）用 `onEdt { }`。
   - 暂停/单步/结束是异步事件、栈帧/变量是异步计算 —— 一律用 support 里现成的协程桥**挂起等待**（带超时上界），不要自旋或 `Thread.sleep`。
2. **多会话**：状态/执行/数据类工具都接受可选 `session` 参数（id 来自 `list_sessions`）。
   解析统一走 `DebugSessionSupport` 的 `resolveSessionOrNull` / `require*Session`：省略 `session` → 取单工程的 `currentSession`。
   新工具的 schema 用 `putSessionParam()` 加该参数。

**新增 MCP 工具**：在对应的 `register*Tools()` 里 `server.addTool(...)`，参数解析后调用顶层 support 函数；
可预期失败抛 `ToolException`，整体用 `toolResult { }` 包裹。

### 调试工作流 Skill（阶段 5.4，非代码产物）

- **`skills/intellij-debugging/`** — 给 Claude 的独立可安装调试工作流 Skill（**不**绑本仓 `.claude`、不随插件编译）：
  `SKILL.md`（核心闭环 + 19 工具速查 + 按 `code` 决策的错误处理 + 人类在环提示）+ `reference/` 五个渐进披露子文件
  （执行超时轮询 / 错误码对照 / 多工程多会话定向 / 数据检查上限 / 控制台读取）+ `INSTALL.md`（拷到 `~/.claude/skills/` 全局使用）。
  纯对现有 19 工具的高层编排指引，不新增能力。**改工具名 / 参数 / 错误码后须同步更新此 Skill，避免文档漂移。**

### 发布打包与用户文档（阶段 5.5 / 5.6）

文档按「面向谁」分工：根目录 `INSTALL.md`（三段式安装：装插件 → Claude 端配 MCP → 装 Skill）+ `USAGE.md`
（使用总览 + 示例）**面向人**；`skills/intellij-debugging/` **面向 AI 编排**。根目录文档链到 Skill 的
`reference/` 而非复制，避免漂移。

- **`scripts/make-release.ps1`** — 重建 `release/` 发布包：从 `build.gradle.kts` 读版本号，（`-Build` 时先
  `buildPlugin`）把插件 zip + `skills/intellij-debugging/` + `USAGE.md` + `release-README.md`(→`README.md`)
  聚到 `release/debug-mcp-<ver>/` 并打成 `release/debug-mcp-<ver>-bundle.zip`。**全量重建、幂等**。
- **真源 vs 产物**：`release/` 整个是产物，**别直接改**里面的副本。改文档要改真源——用户版安装说明改
  `scripts/release-README.md`，使用说明改根目录 `USAGE.md`，Skill 改 `skills/intellij-debugging/`，
  改完跑一次 `make-release.ps1` 刷新。版本号唯一真源是 `build.gradle.kts` 的 `version`（`plugin.xml` 不硬写
  `<version>`，由 `patchPluginXml` 注入）。
- **打包元数据**：`untilBuild="261.*"` 锁 2026.1 全系列（与平台/字节码连锁约束对齐，宁「拒绝加载」不「加载即崩」）；
  插件图标 `META-INF/pluginIcon.svg`(亮)/`pluginIcon_dark.svg`(暗)。仅本地安装，**不**做 `signPlugin`/`publishPlugin`。

## OpenSpec 工作流

本仓库用 OpenSpec 管理变更：`openspec/specs/` 是各能力（breakpoint-management、execution-control、
data-inspection、session-management…）的当前规格；`openspec/changes/` 放进行中的提案，完成后归档到
`openspec/changes/archive/<date>-<name>/`（含该变更的 `design.md`，是实现决策的权威记录）。
有同名 skill（`openspec-propose` / `openspec-apply-change` / `openspec-archive-change` / `openspec-explore`）驱动该流程。

---
> Source: [xiaopacai88888/debug-mcp-idea](https://github.com/xiaopacai88888/debug-mcp-idea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
