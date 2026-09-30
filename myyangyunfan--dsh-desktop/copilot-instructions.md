## dsh-desktop

> > 本文件给 ZCode agent 用。逐字段接口规范以 `dsh-tauri/contracts/` 五份契约为准，

# AGENTS.md — DSH Desktop 工作区指引

> 本文件给 ZCode agent 用。逐字段接口规范以 `dsh-tauri/contracts/` 五份契约为准，
> 开发流程的完整版见 `dsh-tauri/docs/development.md`（本文件不复制其内容）。

## 仓库是什么

DSH Desktop —— 基于 `@deepseek-ai/dsh`（DeepSeek Harness）的桌面客户端。
**v0.5.0 起主线是 Tauri 2（Rust）**；Electron 壳已退役，但 `dsh-desktop/` 的内核侧
Node 逻辑仍是活代码（Tauri sidecar 直接复用，零重写）。主平台 Windows，
同时镜像 GitHub / Gitee（`main` 双向同步）。

| 目录 | 说明 |
| --- | --- |
| `dsh-tauri/` | 桌面壳主线：`contracts/`（五契约）、`src-tauri/`（Rust 工作区）、`sidecar/`（Node 薄封装）、`ui/`、`scripts/`（stage-payload / smoke-installed） |
| `dsh-desktop/` | 内核侧 Node 逻辑：构建期补丁、自愈、插件同步、余额链、`assets/plugins/`、`assets/agent-presets/`、`vendor/dsh-kernel/`（pin 的离线内核 tgz，**必须入库**） |
| `dsh-desktop/scripts/test/` | 全部 Node 测试（186 个 `*.test.js/.mjs`，另有 `fixtures/`、`ta16-snapshots/`、mock server），单测唯一去处 |
| `.github/workflows/` | `ci.yml`（PR 门禁）、`tauri-release.yml`（tag 发版，唯一发布入口）、`release.yml`（退役 Electron 线，全部 `if: false`） |
| `landing/` `openclaw-dsh-bridge/` `docs/` `research/` | 官网页、微信桥接插件、仓库级配图、调研稿 |

## 常用命令

前置：`cd dsh-desktop && npm ci`（`postinstall` 会自动跑 `install-kernel.mjs` + `patch-deps.js`，
这一步决定 `node_modules/@deepseek-ai/*` 的补丁态，跳过会到处红）。

```bash
# --- Node 侧（工作目录 dsh-desktop/）---
npm test                                       # 全量单测：node --test scripts/test/*.test.js *.mjs
node scripts/check-syntax.js                   # 语法门禁（prepack/predist 同款，模式扫描，node --check 抓不到）
node scripts/compat/validate-pin.js            # kernel-pin 与离线 tarball 一致性（fail-closed）
node scripts/compat/patch-surface.js verify node_modules/@deepseek-ai ..   # 补丁干预面漂移

# --- Rust / 壳侧 ---
cd dsh-tauri/src-tauri && cargo test --workspace     # Rust 全量（含契约审计测试）
cd dsh-tauri && node --test sidecar/cli.test.js      # sidecar 真机流程（沙箱 home）
cd dsh-tauri/src-tauri/src/app && cargo run          # 开发运行（loading→内核→Web UI）

# --- 打包（win-x64，三步）---
bash dsh-tauri/scripts/stage-payload.sh              # ① payload 暂存（~500MB，fail-fast）
cd dsh-tauri && npx --yes @tauri-apps/cli build \
  --config src-tauri/src/app/tauri.conf.json --target x86_64-pc-windows-msvc   # ② NSIS
bash dsh-tauri/scripts/smoke-installed.sh            # ③ 安装布局冒烟
```

**没有 lint / typecheck / 格式化工具链**（无 eslint、无 tsconfig、无 prettier）——
语法门禁只有 `check-syntax.js` 与测试，没有可用的 lint 脚本，不要凭空发明。

## 架构边界（改代码前必读）

- **crates 不依赖 tauri 运行时**：`shell-core` / `kernel-process` / `bridge` / `fence` /
  `preview-server` / `session-watcher` / `wsl-backend` 均为纯 std，可独立单测；
  `src/app/` 是装配根，**只接线不实现**。
- **Node 逻辑全部活在 `dsh-desktop/scripts/`**，`dsh-tauri/sidecar/cli.js` 只是薄封装
  （stdout 末行单个 JSON，日志走 stderr）。修内核侧行为改 `dsh-desktop/`，别在 sidecar 里重写。
- **契约先行，且机器强制**：`dsh-tauri/contracts/` 五份文件是接口唯一事实源
  （`bridge-api.md` / `ipc-commands.md` / `data-flow.md` / `plugin-contract.md` / `error-codes.md`）。
  `lib.rs` 的契约审计测试要求「注册命令 ⊆ 契约表」，**加桥命令不改契约 = 测试红**。
  五步流程见 `development.md` §4；加伴随插件见 §5（登记进
  `scripts/lib/companion-plugins.js` 的 `COMPANION_PLUGINS`，id 必须与插件 `cordis.patch.yml`
  的 loader id 一致）。
- **补丁系统**：`dsh-desktop/scripts/lib/patch-registry.js` 是 PatchSpec 唯一清单，
  patch-runner 与健康预检共用同一数据源。它有计数哨兵测试
  （`ta6-registry-invariants`、`unit-patch-registry`）——增删补丁要同步更新哨兵，
  否则测试红。补丁 marker 必须与 transform 的 `already` 判定同源。

## 编码与流程约定

- **中文优先**：文档、commit message、代码注释一律中文（英文版只有 `README.en.md`）。
- **commit**: `<type>: <简述>（#issue号）`，type ∈ `feat/fix/refactor/perf/docs/test/chore/build`；
  分支 `feature/` `fix/` `refactor/` `docs/`；维护者 squash merge，一次提交只做一件事。
- **功能新增 / bug 修复必须带测试**：纯函数用 `scripts/test/unit-*.test.js`，
  bug 回归用例头部注明 issue 号。桌面崩溃/自恢复场景进 `ta3-boot-chain.test.js` 或
  `ta13-soak-*.test.js`。
- **测试必须隔离**：一律用临时目录重定向 `DSH_HOME` / `DSH_TAURI_USERDATA`，
  **绝不触碰真实 `~/.dsh` 与 `%APPDATA%\DSH Desktop`**。
- 可单测纯函数收敛到 `scripts/lib/`，网络与文件编排留在调用方。
- **内核版本 pin 在 `scripts/compat/kernel-pin.json`（exact，禁止浮动）**，
  `vendor/dsh-kernel/*.tgz` 随库提交；换版 = 显式改 pin + 重跑适配器判定 + 全量测试。
- 临时文件（`.tmp-*`、`_*.js`、`*.log`、`portable*/`）已在 `.gitignore` 中，**不要提交**。

## 已知坑

- **发版唯一入口是推 `v*` tag** → `tauri-release.yml`。`release.yml` 是退役 Electron 线，
  历史上写 `false && A || B` 造成过「假短路」，别去复活它。流程见 `.github/RELEASE_RUNBOOK.md`。
- **版本号要三处同步**：`dsh-desktop/package.json`、`dsh-tauri/src-tauri/Cargo.toml`
  （`workspace.package.version`）、`dsh-tauri/src-tauri/src/app/tauri.conf.json`。
  其中 `tauri.conf.json` 的 version 会被 CI 与 tag 做 fail-fast 比对，漏改直接发版失败。
- **冒烟测试绝不跑真安装器**：NSIS 的 PREINSTALL 会静默卸载本机真实版本。
  冒烟用手拼安装布局 + `DSH_HOME`/`DSH_TAURI_USERDATA` 隔离。
- **NSIS 钩子（installerHooks.nsh）改动必须过 `makensis` 编译验证**——宏展开、栈平衡、
  `/SD` 参数位置都曾导致安装器卡死或编译阻断。
- **关窗 ≠ 退出**：`closeToTray` 缺省 true（`src/app/src/windows.rs` 的 `CloseRequested`），
  点 × 只是隐藏到托盘、**内核 node 继续跑**。于是覆盖安装写 `@img/sharp-win32-x64/lib/
  libvips-42.dll` 会被 Windows 拒绝（不允许覆盖已被加载的映像），用户看到
  「Error opening file for writing」。安装器已加固：`installer-template.nsi` 在
  `CheckIfAppIsRunning` 之后插 `DSH_KILL_TREE_NODES`（按 CIM 的 ExecutablePath/CommandLine
  前缀精确清本安装树的 node）+ `DSH_WAIT_FOR_INSTDIR_RELEASE`（有界等句柄释放 ≤15s）。
  两个宏都在 `installerHooks.nsh`，**别再改回 `$_.Path`**——实测安装器子 PowerShell 里它
  对所有进程都是空的（过滤永远不命中，等于没清理），回归锁在 `ta14-upgrade-dirty-home`。
  手动验证这两个宏：`makensis -INPUTCHARSET UTF8` 编译一个 `!include` 真钩子文件的独立
  脚本，用 `/D=<临时目录>` 跑 `/S`，别在真安装目录上试。
- **稳定性三原则（评审默认立场）**：① 客户端必须能打开，装配失败终态恢复页而非退出；
  ② 兼容性不报错，意外以日志收场（`panics.log`）不以崩溃收场；③ 用户数据不动。
- `unit-updater` 的两个 fallback 用例在依赖装好时显示 `skip`，属**预期**而非失败。
- 文档里的测试基线数字常滞后（如 `development.md` 写 177 Rust 例 / 71 文件 Node，
  实际 `scripts/test/` 已有 186 个测试文件）。**以实测输出为准**。
- 部分文件含 GBK 遗留注释（如 `dsh-tauri/src-tauri/Cargo.toml`），按 UTF-8 读会显示乱码；
  编辑这类文件时保持原编码，不要顺手「修正」成全角乱码以外的内容。
- 调试开关（随产物保留）：`DSH_TAURI_DIAG=1` 页面探针、`DSH_TAURI_DEVTOOLS=1`、
  `DSH_TAURI_REPO_ROOT=<dir>` 显式内核目录、`DSH_HOME` / `DSH_TAURI_USERDATA` 数据目录重定向。
- **`assets/plugins/dsh-better-sidebar` 的 `src/` 与 `lib/` 已经同源，重建是正常路径**：
  该插件是 vendored 上游包，历史上曾经「`lib/` 领先 `src/`」——`lib/client-editor.js` 里的
  「按变更查看 diff」（`splitLines` / `diffRows` / `diffStats` / `formatHistTime` /
  `DiffTurnsPanel`）是直接手改进产物的，`src/` 没有；`src/client/sidebar.module.css` 另有一段
  vendored 同步截断留下的孤立 `}`。**两处都已在 2026-09 补齐/删除**（补齐见 `src/client/diff-turns.ts`
  + `DiffTurnsPanel.tsx`，删除见该 CSS 里的注释）。现在改行为的姿势是：改 `src/` → 在该目录跑
  它的打包器（tsdown / bundle）→ 用「产物函数集合对等」+ `unit-better-sidebar-*` 测试核对。
  两个频道产物由同一份 `src/client/index.tsx` 编两遍（`lib/client.js` 官方频道 / `lib/client-registry.js`
  插件注册表频道），所以只改 src 就够，**不要只手改其中一个产物**。
- **插件的运行时落点是 repo 的 `assets/plugins/`，不是 profile**：启动期的伴随插件同步会把
  payload 里的插件目录镜像进 `<DSH_HOME>/profiles/<name>/node_modules/`，所以手工往 profile
  里塞文件会在下次启动被覆盖——要改就改 `assets/plugins/`（开发机是仓库目录，安装机是
  `<安装根>/dsh-desktop/assets/plugins/`）。
- **内核右栏是可扩展面，别去改内核包**：`dsh-client-ui-sidebar-right` 的
  `sidebarRightTabs.register` + `sidebar.right.pane.tab` 是公开两段式标签 API（内核自己的
  文件/文档预览/终端也用它）。本仓库的 better-sidebar 已经用它把工作台接成右栏里的一个标签，
  设计与踩坑见 `dsh-desktop/docs/better-sidebar-kernel-integration.md`。

## 动敏感区域前先读

| 要改什么 | 先读 |
| --- | --- |
| 桥命令 / IPC / 页面 API | `dsh-tauri/contracts/` 五契约 + `docs/development.md` §2 §4 |
| 打包 / 发版 / 更新链 | `.github/RELEASE_RUNBOOK.md`、`dsh-tauri/docs/release-keys.md`、`docs/development.md` §6 §7 |
| 补丁 / 自愈 / 插件同步 | `dsh-desktop/scripts/lib/patch-registry.js` 头部注释、`dsh-desktop/docs/plugin-center-architecture.md` |
| 余额链 | `dsh-desktop/docs/balance-architecture.md` |
| WSL 模式 | `dsh-desktop/docs/wsl-verification.md`、`dsh-tauri/sidecar/wsl-*.js` |
| 任意改动前的变更史 | `dsh-desktop/CHANGELOG.md`、`dsh-tauri/CHANGELOG.md`（逐提交实测记录） |
| PR 流程与测试硬性要求 | `CONTRIBUTING.md` |

---
> Source: [myYangyunfan/dsh_desktop](https://github.com/myYangyunfan/dsh_desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
