## android-basic

> - 公共 API 与行内注释使用中文（KDoc 规范见 `android-code-style` skill）

# AGENTS.md — Android-Basic

## 基础规则
- 始终使用简体中文回答
- 公共 API 与行内注释使用中文（KDoc 规范见 `android-code-style` skill）

## 模块
- `app/` — 演示模块，依赖 `:Basic`
- `Basic/` — 库模块，发布到 JitPack（坐标 `com.github.mstains:android-basic:1.0`，见 `Basic/build.gradle:8-13`）
- 两者共享 `namespace 'com.letter.basic'`（历史原因，勿随意改动）
- 扩展函数统一放在 `Basic/src/main/java/com/letter/basic/extend/`

## 构建命令
- `./gradlew assembleDebug` / `assembleRelease`
- `./gradlew :Basic:assembleRelease`（产出 AAR + sources jar，供 JitPack 发布）
- `./gradlew test` — 仓库内只有占位测试，等同空跑；提交前请用本地编译验证
- `./gradlew clean`

## 工具链
- Gradle 8.10.2 / AGP 8.8.2 / Kotlin 2.2.10 / compileSdk 35 / minSdk 23
- JVM target：两模块均为 Java 21（`compileOptions` + `kotlinOptions.jvmTarget`）
- `coreLibraryDesugaring`：两模块均启用 + `desugar_jdk_libs:2.1.4`（添加新模块时不要遗漏）
- 仓库镜像：腾讯云 + 阿里云（`settings.gradle:11-16, 28-33`）

## 依赖管理
- 所有版本号在 `gradle/libs.versions.toml` 集中维护
- 子模块 `build.gradle` 通过 `libs.xxx` 引用，禁止直接写坐标

## 静态分析 / 注释检查
- `:Basic` 模块配置了 **detekt**（Kotlin KDoc 检查，规则见 `.detekt.yml`）和 **Checkstyle**（Java Javadoc 检查，规则见 `checkstyle.xml`），均仅启用注释规则
- `./gradlew :Basic:checkComments -Pscope=full` — 统一执行 detekt + checkstyle，产出 JSON 报告到 `Basic/build/reports/comments/check-comments.json`
- detekt 默认排除 `override` 方法（在 `checkComments` task 中过滤）；Checkstyle 依赖 `@Override` 注解排除
- `app` 模块未配置任何静态分析

## 代码层硬规则
- 继承 `BaseMultiStateVBActivity` 时，`onCreate` 固定顺序为
  `initStatusBar → initView → initData → initListener`
  （见 `Basic/.../activity/BaseMultiStateVBActivity.kt:29-36`），子类不要重复调用 super
- `IntentExt.kt` 中 `baseStartActivity*` / `baseStartActivityForResult*` 已 `@Deprecated`，
  新代码改用 `ResultCallbackLauncher` 系列（`Basic/.../extend/ResultCallbackLauncher.kt`）
- `BaseMultiState*` 自动入栈 `ActivityController`；若需全局 `Application`，
  在自家 `Application.onCreate()` 中调用 `ActivityController.setApplication(this)`
- `BroadcastUtil` 基于已废弃的 `LocalBroadcastManager`，仅作兼容保留
  （见 `Basic/.../utils/BroadcastUtil.kt`）

## 新增能力的归位
- 权限 / 相册 / 拍照等 `ActivityResultLauncher` → `ResultCallbackLauncher.kt`
- 日期 `DateTimeFormatter` 模板 → `Basic/.../manager/AppDateFormatter.kt` / `Basic/.../manager/DatePatterns.kt`
- 权限中文名 → `ChinesePermission` 枚举 + `toPermissionChineseName*` 扩展

## 验证 / CI
- 仓库内无 lint 配置、无 CI 工作流、无有意义的测试
- 提交前只能依赖本地编译；CI 流水线由调用方自行配置

## 注释规范
正文见 opencode 全局 skill `android-code-style`。
速记：public API 必带 KDoc；行内注释回答"为什么"而非"做什么"；TODO 格式
`// TODO(作者/issue号): 原因 → 计划方案`。

<!-- CODEGRAPH_START -->
## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->

---
> Source: [mstains/Android-Basic](https://github.com/mstains/Android-Basic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
