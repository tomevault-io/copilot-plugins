## pi-desktop

> pi-desktop 是一个面向本地开发工作的 Electron 桌面应用，用于在多个项目目录之间管理和运行 pi RPC Agent。应用提供多项目工作区、会话时间线、历史会话恢复、文件抽屉、Git 分支信息、模型选择、工具调用展示以及打包发布能力，目标是让用户可以在桌面端更稳定地管理多个 pi 编码助手会话。

# AGENTS.md

## 项目简介

pi-desktop 是一个面向本地开发工作的 Electron 桌面应用，用于在多个项目目录之间管理和运行 pi RPC Agent。应用提供多项目工作区、会话时间线、历史会话恢复、文件抽屉、Git 分支信息、模型选择、工具调用展示以及打包发布能力，目标是让用户可以在桌面端更稳定地管理多个 pi 编码助手会话。

## 开发要求

- 修改核心逻辑、复杂状态流转、业务规则、数据转换或异常处理时，需要补充有价值的代码注释。
- 注释应说明为什么这样做、对应的业务规则或边界条件，不要逐行解释显而易见的代码。
- UI 调整应尽量保持现有桌面三栏布局和微信式交互风格，避免引入无关重构。
- 修改后应根据影响范围运行必要验证，例如 `npm run typecheck`。

## 发版要求

发版或准备 release 时，必须核对并更新以下内容：

1. `README.md` / `README.zh-CN.md`
   - 核对功能说明、截图说明、安装/使用说明是否仍然准确。
   - 如果本次版本包含用户可见的新功能、行为变化或配置变化，需要同步补充说明。

2. `CHANGELOG.md` / `CHANGELOG.zh-CN.md`
   - 为新版本增加对应版本号和日期。
   - 用简洁条目记录新增、优化、修复等用户可感知变化。
   - 中英文更新日志应保持信息一致。

3. GitHub Release 说明
   - 发布时需要在 release notes 中写明本次版本的主要变化。
   - Release 说明应覆盖 README 和 CHANGELOG 中提到的关键用户可见调整，避免只写版本号或空说明。

4. 版本号
   - 核对 `package.json` 和 `package-lock.json` 中版本号一致。
   - 发版提交应清晰标识版本，例如 `chore: release vX.Y.Z`。

---
> Source: [ayuayue/pi-desktop](https://github.com/ayuayue/pi-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
