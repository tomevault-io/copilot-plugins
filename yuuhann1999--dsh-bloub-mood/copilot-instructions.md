## dsh-bloub-mood

> bloub mood 心情图标插件（v2.2.0）：把 DSH web 端 logo/状态点换成 bloub 心情形象，

# AGENTS.md — dsh-bloub-mood 本地开发指南

bloub mood 心情图标插件（v2.2.0）：把 DSH web 端 logo/状态点换成 bloub 心情形象，
并内置「桌面启动」子功能。开发工作区（上游：github.com/Yuuhann1999/dsh-bloub-mood，npm: dsh-bloub-mood）；改动后按下方流程重新构建安装。

## 结构

- `client-template.js` — 浏览器半边源码（表情系统 + 设置页 UI）。**唯一需要手改的前端文件**
- `lib/client.js` — 构建产物（`node build.mjs` 生成，勿手改；内嵌 128 SVG + 40 GIF，约 13MB）
- `lib/index.js` — 宿主半边入口：物化桌面入口 + 注册 `/desktop-entry/*` 路由
- `lib/desktop-entry.mjs` — 桌面启动核心（mac/win 双平台物化、开关/图标 API）
- `assets-expressions/` `assets-scripts/` — 表情素材（构建时内嵌进 client.js）
- `assets-appicons/` — 桌面入口图标：`AppIcon.{classic,joy,excite}.icns`（mac）+ 同名 256px PNG（win 转 ico + 设置页预览）
- `mac/main.swift` — macOS 原生壳源码（WKWebView 独立 App 进程；物化时用 xcrun swiftc 编译进 .app，无 CLT 则回退 bash+浏览器）
- `build.mjs` — 把素材内嵌进 template 生成 `lib/client.js`
- `cordis.patch.yml` — 挂载行 + `config.desktopEntry` 默认值（enabled/icon/appName/windowSize）

## 桌面启动子功能（v2.2.0，原 dsh-desktop-entry 插件合并而来）

- mac：`~/Applications/DSH Desktop.app` = **原生壳**（编译自 mac/main.swift 的独立进程，Dock 自带图标；实例配置在 Info.plist Dsh* 键；源码 hash 变更才重编译，marker `.native-<hash>`）；win：开始菜单 `.cmd` + 桌面 `.lnk`（PNG→ICO + PowerShell WScript.Shell，仍为浏览器窗口，原生壳待 WebView2）
- 启动器逻辑：后端在线复用 → 离线 `nohup dsh web`（win `start /B`）→ 轮询就绪 → Chromium `--app` 无栏窗口；日志 `~/.dsh/logs/desktop-entry.log`
- 图标三选一（bloub 形象）：classic=cercle·neutre（默认）/ joy=galet·hilare / excite=capsule·excite
- 持久化：`$DSH_HOME/dsh-bloub-mood/desktop-entry.json`，优先级 持久化 > config > 默认
- API：`GET /desktop-entry/api/state` · `POST .../enabled` `{enabled}` · `POST .../icon` `{icon}` · `GET /desktop-entry/icon/:id.png`
- 关闭 = 移除入口文件；重启后尊重持久化状态，不会擅自重建

## 开发循环

```sh
node build.mjs                                        # 1. 改 client-template.js 后构建
cd ~/.dsh/profiles/web && dsh plugin --profile web remove dsh-bloub-mood
cd /Users/zeng/Documents/Project/harness/dsh-bloub-mood && dsh plugin --profile web add "file:$PWD"
pkill -f "dsh web" && open ~/Applications/DSH\ Desktop.app   # 3. 重启后端（.app 会冷启动它）
```

注意：`file:` 安装是硬链接，remove→add 才会刷新已改文件；只 add 显示 "Already up to date" 是正常的。

## 约定

- **设置页文案极简**：一句话引导即可，不写机翻式长注释；单行「标题 + 开关」布局，
  不出现重复的状态文案（不要「桌面启动 · 已启用」这种叠加）；成功操作零提示，只显示 ⚠️ 错误。
- **i18n（v2.3.0）**：全部 UI 文案走客户端 `locale` 服务（inject `["slots","locale"]`，
  `ctx.locale.register(LOCALE_NS,{zh,en})` + `bind`），禁止中英混排的「中文 / English」标签；
  设置项注册带 `locale: LOCALE_NS`，section `label` 用 `() => t("nav")` 函数形式（切换语言实时生效）；
  新增文案必须同时在 ZH_DICT / EN_DICT 补齐。
- **品牌位（DSH ≥ 0.1.1）走官方 Slot**：`sidebar.brand.mark`（bloub 动图）与
  `sidebar.brand.name`（字标+徽章）注册渲染，React 原生抗重渲染；旧版 0.1.0 无此
  Slot 自动落到 whale-clip SVG 手术回退。改品牌相关逻辑两处都要兼顾。
- **第三方整栏替换侧边栏时**（如 dsh-oil-creator 以 priority -1 接管整个 sidebar 槽，
  其 .oilBrand 品牌区硬编码、不渲染 sidebar.brand.mark 插槽）：Slot 通道失效，走
  replaceOilBrand DOM 手术（藏其图标/文字 + 注入动图/字标，MutationObserver 自愈，
  setHostStyle 账本保证停用还原，注入 img 在 [data-slot] 内不受 sweepOrphans 误删）。
- 图标预览必须用 `/desktop-entry/icon/:id.png`（真实生成图），保证所见即桌面所得。
- 换图标源图后：重跑 `scripts/build-icon.sh`（在已删除的 dsh-desktop-entry 仓库；如无则用
  sips+iconutil 从 1024px PNG 生成各尺寸 icns + 256px PNG）。
- 宿主半边只依赖 `webServer`（inject 声明）；不引第三方依赖。
- 语义化版本：功能变更 bump package.json + 代码内 `version` 常量（desktop-entry.mjs 两处）。

---
> Source: [Yuuhann1999/dsh-bloub-mood](https://github.com/Yuuhann1999/dsh-bloub-mood) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
