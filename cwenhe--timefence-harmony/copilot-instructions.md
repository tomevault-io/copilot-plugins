## timefence-harmony

> - 本仓库是 HarmonyOS 6.1/API 23 的 ArkTS + ArkUI Stage 工程，与 `timefence-android` 完全独立。

# TimeFence Harmony 协作规则

## 项目边界

- 本仓库是 HarmonyOS 6.1/API 23 的 ArkTS + ArkUI Stage 工程，与 `timefence-android` 完全独立。
- 本项目不是 APS/SCH 排程项目，不引入 `aps-java` 产品依赖。
- 全局悬浮窗必须使用官方 `window.WindowType.TYPE_FLOAT` 和 `ohos.permission.SYSTEM_FLOAT_WINDOW`；不得用应用内浮层冒充跨应用悬浮。

## 默认分支和验证

- 默认开发分支为 `develop`。
- 修改 `.ets`、`.json5` 或文档后运行：
  - `node scripts/clock-formatter-test.mjs`
  - `node scripts/validate-project.mjs`
  - `git diff --check`
- 有 DevEco Studio 时额外运行 `./hvigorw assembleHap --mode module -p module=entry`，并在 API 23 真机验收权限和窗口行为。

## 代码约定

- 新增函数必须有中文职责注释，说明关键副作用或边界。
- 主页面不直接操作 `window.Window`，统一通过 `FloatingClockService`。
- 任何权限或设备能力失败都要转成用户可理解的状态，不得静默吞错。
- 不提交证书、profile、`build/`、`oh_modules/`、日志或真机快照。

## 文档和提交

- 文档、注释和提交说明使用简体中文；提交遵循 Conventional Commits，例如 `feat: 添加鸿蒙跨应用悬浮时钟`。
- 需求级验收场景同步维护在设计文档和 `docs/project-integration/testing.md`。

---
> Source: [cwenhe/timefence-harmony](https://github.com/cwenhe/timefence-harmony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
