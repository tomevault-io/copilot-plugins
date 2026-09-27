## oi-skill

> 本仓库包含 OI.SKILL Agent Skill，以及一个 Electron 桌面端学习工作区。

# OI.SKILL 项目规则

## 定位

本仓库包含 OI.SKILL Agent Skill，以及一个 Electron 桌面端学习工作区。

## 运行与验证

- Skill 安装：`npx skills add https://github.com/FlashingChen/oi.skill/tree/main/skill`
- 桌面端开发：`cd app && npm install && npm start`
- 桌面端门禁：`cd app && npm test && npm run lint`
- 本地打包：`cd app && npm run package`（macOS dir 产物）；Windows installer 使用 `npm run package:win`

## CI/CD（GitHub Actions）

- 工作流：`.github/workflows/build.yml`，触发条件：push 到 `main`、`v*` tag、任意 PR、手动 `workflow_dispatch`。
- 流程：`test` job（ubuntu，`npm test` + `npm run lint`）通过后，三个 `package` job 并行构建：
  - macOS：`electron-builder --mac dmg zip --x64 --arm64`（macos runner）
  - Windows：`electron-builder --win nsis --x64`（windows runner）
  - Linux：`electron-builder --linux AppImage --x64`（ubuntu runner）
- 产物以 `oi-skill-mac/win/linux` 命名上传为 workflow artifacts，每次运行都可下载。
- 打 `v*` tag 时额外发布 GitHub Release，自动附带三端产物。
- CI 不签名（`CSC_IDENTITY_AUTO_DISCOVERY=false`），产物均为 unsigned。
- 改动打包目标/架构时，同步更新本工作流的 `matrix.args` 与 `app/package.json` 的 `build` 配置（当前单一事实源在 workflow 的 matrix）。

## 技术栈

- Skill：Markdown、YAML、Python 对拍脚本。
- Desktop：Electron、Node.js、内置 `@earendil-works/pi-coding-agent` SDK、原生 IPC。

## 目录与约定

- `skill/` 是可分发 Skill 的权威目录；改动 Skill 行为先改这里。
- `app/` 是桌面端源码、测试和打包配置；用户运行数据默认在 `~/.oi-skill/`，不提交到仓库。
- `README.md` 说明用户安装和使用；`app/README.md` 说明桌面端运行与打包。
- API key 只留在本地私有配置或环境变量中，不写入 session、日志或文档。

## 当前状态与下一步

- 当前分支为 `agent/relocate-skill-files`；桌面端代码已在 `app/`，不是旧的 `apps/desktop/` 路径。
- 2026-08-05 当前本地验证：app 测试 26 项通过，Node 语法检查通过；本地 macOS/Windows 产物已存在，Windows `app.asar` 与当前主进程源码一致，macOS 包为旧产物。
- 本次重新执行 `npm run package` 因 electron-builder 的网络请求被中止而失败；现有产物是此前生成的，不能视为本次打包成功。
- 发布状态仅能确认 local artifacts；未在本项目中验证 merged、deployed 或 live surface。
- 后续改动完成后先运行 app 测试与 lint，再按需重新打包，并同步 `README.md` / `app/README.md` 的现役说明。
- 三端自动构建已由 GitHub Actions 接管（见上方 CI/CD 节）；本地 `npm run package` 仍用于快速验证 macOS 产物。

---
> Source: [FlashingChen/oi.skill](https://github.com/FlashingChen/oi.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
