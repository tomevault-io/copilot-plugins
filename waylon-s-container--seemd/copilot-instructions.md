## seemd

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

seemd：本地、**默认断网**、功能极简的 Markdown 查看/编辑器。Tauri 2 + WebView2，Windows 桌面应用（v1 明确不做跨平台）。查看为主 + CodeMirror 6 切换式轻编辑（非分栏）。无框架——前端是 vanilla TypeScript 模块，后端是少量 Rust。

## 常用命令

```bash
npm install                  # 安装前端依赖
npm test                     # 前端全部测试（Vitest + jsdom）
npx vitest run tests/sanitize.test.ts        # 运行单个测试文件
npx vitest run -t "strips"                   # 按名称过滤
npm run test:watch           # 监听模式
npm run build                # tsc（严格模式）+ vite build —— headless 下的编译验证手段
npm run tauri dev            # GUI 开发模式（需要人眼，headless 跑不了）

# Rust 侧（git-bash 里必须先 export PATH="$HOME/.cargo/bin:$PATH"，PowerShell 不用）
cd src-tauri && cargo test   # Rust 单元测试
cd src-tauri && cargo check  # 快速编译检查（比 test 快，无 GUI 验证时的替代）
cd src-tauri && cargo test validate   # 运行单个 Rust 测试（按名过滤）

npm run tauri build          # release exe + MSI（MSI 需 WiX，国内网络下载易超时，见 README）
```

## 本机环境要点

- **cargo 不在 git-bash PATH 上**（`~/.cargo/bin`，bash 不加载 profile）：所有 Rust/tauri 命令前先 `export PATH="$HOME/.cargo/bin:$PATH"`。
- **仓库根目录 `.cargo/config.toml` 配了 rsproxy 国内镜像**（crates.io 在本网络不可用）。不要删除；首次全量构建后 `target/` 已缓存，后续增量快。
- **npm 12+ 安装需镜像源**：lockfile 的 resolved 全指向 npmmirror，默认官方源报 `EALLOWREMOTE`。用 `npm install --registry=https://registry.npmmirror.com`。
- **Tauri 打包要联网下载（WiX / WebView2 离线包）时必须设代理环境变量**：本机代理 `127.0.0.1:7890` 只配在 Windows 系统层，Tauri 的 Rust 下载器不读系统代理——`$env:HTTPS_PROXY='http://127.0.0.1:7890'`（HTTP_PROXY 同设）再 build，否则 `timeout: global`。
- **禁止用裸 `cargo build --release` 产出用于验证/分发的 exe**：没有 Tauri CLI 设置的环境变量时，tauri-build 按 dev 配置编译，窗口指向 `http://localhost:1420`——本机实测窗口空白、前端 JS 一个字都不执行（关闭行为、探针全部缺席），会得出完全错误的"验证通过"结论。验证/分发一律 `npm run tauri build`；`cargo check`/`cargo test` 不受影响。
- **前端调用 window/webview 等核心插件命令前查 capabilities**：`core:default` 只含各核心插件的默认集（window 默认集只有 getter）。X 关窗曾因 `destroy()` 缺 `core:window:allow-destroy` 被权限层拒绝而失效（2026-08-24 已修）。
- **Headless 无 GUI**：任何"跑起来看看"的验证用 `cargo check` / `cargo test` / `npm test` / `npm run build` 替代；GUI 走查项如实记为 human-deferred，不得声称已验证。

## 架构

### 渲染管线（顺序是不变量）

`src/main.ts` 的 `renderActive`：`composeHtml(text)` → `renderMermaid(html, theme)` → `rewriteLocalImages(html, baseDir)` → `sanitize(html, online)` → `IframeViewer.setHtml(...)`。

- **`sanitize` 永远是最后一步**：所有渲染器输出（含 Mermaid 的 SVG、本地图片改写结果）都必须经过 DOMPurify 再进 iframe。新增渲染步骤要插在 sanitize 之前。
- `composeHtml`（`src/render/pipeline.ts`）：数学先挖占位（防 marked 破坏）→ marked → Prism；`language-mermaid` 代码块原样传给 mermaid 阶段。
- Mermaid **懒加载**：无图块时不加载库。

### 安全模型（HARD 约束，三层冗余）

1. DOMPurify 净化（`src/render/sanitize.ts`）：断网模式下剥离远程 `src`/`href`/`srcset`（逗号逐项）/`style url()`/`xlink:href` 等。曾有两个断网绕过（srcset、style url()）被整体 review 抓出并修复 + 回归测试——改动此处必须看这些测试。
2. 沙箱 iframe：`sandbox="allow-same-origin"`（属性声明在 `index.html`，`src/viewer/iframe-viewer.ts` 注入 srcdoc），**不带 `allow-scripts`**。
3. CSP（`src-tauri/tauri.conf.json`）：`script-src 'self'` 永久严格。

**默认断网是内核约束**：不主动发起任何网络请求；工具栏 Online 开关只放宽图片（sanitize 的行为），**永不**放松脚本限制或 CSP。

### IPC 信任边界

前端从不直接信任路径。所有路径来源（对话框 / 拖拽 / 单实例 / 启动参数 / 状态恢复）都汇入 `src/io/bridge.ts` 的 `readFile`/`writeFile` → Rust `validate_md_path`（强制绝对路径 + `.md`/`.markdown` 扩展，在任何 fs 操作之前）。新增 IPC 命令必须在 Rust 侧校验入参。

### 冷启动 vs 二次打开（.md 关联）

两条路径不同，勿混用（详见 `src-tauri/src/lib.rs` 注释）：
- **冷启动**（双击 .md 启动进程）：argv 存入 Rust managed state（`StartupFile`），前端启动后用 `get_startup_file` 命令拉取——直接 `emit("open-file")` 会因前端监听器未挂上而丢失（Tauri 不为迟到订阅者排队事件）。
- **热路径**（已运行时再双击）：single-instance 插件回调 `emit("open-file")`，前端 `listen` 已挂上，正常收到。

### 本地相对图片（seemd-asset 协议）

沙箱 srcdoc iframe 无 base URL，相对 `./img.png` 解析不到 .md 所在目录。`src/render/local-images.ts` 把相对 `<img src>` 改写为 `http://seemd-asset.localhost/<enc(baseDir)>/<enc(rel)>`；Rust `src-tauri/src/asset.rs` 的协议 handler 只放行图片扩展名并从磁盘读文件服务。

### 状态持久化

- 应用状态（lastFile、scrollByPath、theme、online）：`state.json`（app_data_dir），经 `read_app_state`/`write_app_state` 命令读写（`src/state/persistence.ts`）。
- 窗口几何：`tauri-plugin-window-state` 自动处理。
- 关窗 flush（`src/main.ts` 的 `onCloseRequested`）：`preventDefault()` → `await saveState` → 显式 `destroy()`。不 preventDefault 的话，失败的 invoke 会让 X 按钮卡死窗口（曾出过此 bug）。

### 编码

读取：BOM（UTF-8/UTF-16）→ 无 BOM 则 chardetng 嗅探（兼容 GBK）；保存：一律写 UTF-8（`src-tauri/src/encoding.rs`）。

### 容易踩的坑

- **iframe CSS 内联**：KaTeX/Prism CSS 在 `iframe-viewer.ts` 用 Vite `?raw` 导入、拼进 srcdoc。`main.ts` → `iframe-viewer` 的导入链断了会被 tree-shake 掉，iframe 里样式全丢。
- **脏标记保护**：CM6 的 updateListener 对程序化 `setValue` 也触发；`main.ts` 用 `loading` 标志包住每次程序化写入，否则刚打开的文件就被误标 dirty。改动编辑器装载逻辑时保留该守卫。
- IPC invoke 的 JS 参数名与 Rust 命令签名的对应（Tauri 自动做 camelCase↔snake_case 转换），新增命令时两侧对照。

## 工作流约定

- **SDD 流程**：计划在 `docs/superpowers/plans/2026-08-06-seemd-markdown-viewer.md`；进度台账在 `.superpowers/sdd/progress.md`——完成任务后更新台账（含 review 结论、human-deferred 项）。台账比 README 的「待完成」列表新（本地图片、Ctrl+S、错误横幅、主题修复、CSP 加固等 fast-follow 已实现，README 未同步）。
- **实现不落在 master**：在 feature 分支（如 `feat/viewer-v1`）上工作。
- **提交信息**：conventional commits 带scope，如 `feat(render): ...` / `fix(viewer): ...` / `build: ...` / `docs(sdd): ...`。
- **测试哲学**：纯逻辑（pipeline、sanitize、tab-manager、bridge 辅助函数、encoding、validation、asset 解析）全部单测覆盖；GUI 行为 headless 无法验证，记 human-deferred。
- 已知未修的小问题清单在 README「待完成 / 待验证」与台账的 DEFERRED Minors——区分"新 bug"和"已记录的已知项"。

---
> Source: [Waylon-s-container/seemd](https://github.com/Waylon-s-container/seemd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
