## vibetalking

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么

VibeTalking —— 一个 Tauri 2 写的菜单栏 push-to-talk 听写工具,支持 macOS + Windows。按住全局快捷键(macOS 是 Fn,Windows 是右 Alt)说话,松开后通过字节 OpenSpeech 转写,自动粘到当前焦点的输入框。

仓库根目录还有一些不相关的 Python 脚本(`transcribe.py`、`format_*.py`、`*.m4a/.aac/.txt`)和 `old/`(已归档的 Swift 原型)。真正的 app 代码全在 `app/` 下。

## 常用命令

都在 `app/` 下跑:

```bash
npm install
npm run tauri dev      # 开发:vite + Rust 热重载
npm run tauri build    # 出包 → src-tauri/target/release/bundle/
npm run build          # 只构建前端(vite,输出到 ../dist)
```

只动 Rust:

```bash
cargo check --manifest-path app/src-tauri/Cargo.toml
cargo build --manifest-path app/src-tauri/Cargo.toml --release
```

没有自动化测试,验证靠跑应用试快捷键。

## 本地测试构建产物

`tauri build` 出包后,产物落在:

```
app/src-tauri/target/release/bundle/
├── dmg/VibeTalking_<ver>_aarch64.dmg     # macOS 安装镜像
└── macos/VibeTalking.app                 # 可直接双击的 .app
```

本机当前安装位置:

```
/Applications/VibeTalking.app               # 当前(新名)
/Applications/VibeTalk.app                  # 旧名,可删
```

启动:`open /Applications/VibeTalking.app`,然后看菜单栏麦克风图标。日志走 stderr;开发态可以直接 `RUST_LOG=info npm run tauri dev` 看。

替换本机 `/Applications/VibeTalking.app` 后,一定要确认 macOS 签名 identifier 稳定。否则系统辅助功能权限会绑定到临时签名身份,设置页可能看起来已授权,但 `CGEventTap` 收不到 Fn,表现为按住 Fn 托盘不变橙色、没有录音状态:

```bash
codesign -dv --verbose=4 /Applications/VibeTalking.app 2>&1 | grep Identifier
```

期望输出里是 `Identifier=com.vibetalk.dictation`。如果看到类似 `Identifier=voice2text-...` 的随机值,先退出应用并重签:

```bash
pkill -f '/Applications/VibeTalking.app/Contents/MacOS/voice2text' || true
codesign --force --deep --sign - --identifier com.vibetalk.dictation /Applications/VibeTalking.app
tccutil reset Accessibility com.vibetalk.dictation
open /Applications/VibeTalking.app
```

重置后需要在 macOS 系统设置里重新允许 VibeTalking 的“辅助功能”权限,然后再测试 Fn。不要只看设置页文字,最终验证标准是按住 Fn 时托盘图标立刻变橙色。Fn/Globe 在 macOS 上可能被系统设置为“切换输入法”;如果第一次能录音、后续变成切换输入法,说明系统行为没有被拦截,优先检查 `hotkey/macos.rs` 里 event tap 是否用了可拦截的 `CGEventTapOptions::Default`,并且对 Fn/Globe 相关事件返回 `CallbackResult::Drop`。

### 运行时数据目录(用户配置 + 历史)

```
~/Library/Application Support/com.vibetalk.dictation/
├── settings.json     # 字节 OpenSpeech 凭证 + 语言
└── history.json      # 最近 500 条转写记录
```

首次启动 `SettingsStore::load()` 会用 `config.rs` 里 hard-code 的默认值生成 `settings.json`,所以本机不需要再手动填 key。如果改了 `config.rs` 的 default 想生效,要么删掉这个 `settings.json` 让它重建,要么直接编辑该文件。

旧 bundle id 残留的 `~/Library/Application Support/com.voice2text.app/` 可以删。

### 权限重置(macOS 调试用)

换 bundle identifier 或者签名后,辅助功能/麦克风的授权会失效。强制重置:

```bash
tccutil reset Accessibility com.vibetalk.dictation
tccutil reset Microphone    com.vibetalk.dictation
```

下次启动会重新弹授权框。

如果刚替换过 `/Applications/VibeTalking.app`,先检查并修正上面的 `codesign` identifier,再做 `tccutil reset`。否则会反复授权到错误/临时签名身份,Fn 热键仍然没反应。

## 发布流程

推一个 `v*` 形式的 tag(例如 `v0.1.0`),`.github/workflows/release.yml` 会用 `tauri-apps/tauri-action@v0` 矩阵构建三份产物(macOS aarch64、macOS x86_64、Windows)并上传到 draft release。然后 `gh release edit <tag> --draft=false` 发布。

要为同一个版本重新触发构建:删掉 release 和 tag(`gh release delete v0.1.0 --cleanup-tag --yes`),再推一次。`app/src-tauri/tauri.conf.json` 里的 `version` 要和 tag 对齐。

workflow 里的 `permissions: contents: write` 是 GITHUB_TOKEN 创建 release 必需的;同时仓库默认 workflow 权限也必须改成 "read and write":

```bash
gh api -X PUT /repos/<owner>/<repo>/actions/permissions/workflow \
  -f default_workflow_permissions=write
```

## 架构

### 事件主链路

`hotkey::spawn_listener` 跑在一条独立线程上,往 `mpsc::UnboundedSender` 里推 `HotkeyEvent::{Pressed,Released}`。`lib.rs` 的 `spawn_event_loop` 又在自己的线程上消费这些事件:

- **Pressed** → `Recorder::start()`,emit `recording-state=true` 给 webview,托盘图标换橙色
- **Released** → `Recorder::stop()` 返回 `(wav_bytes, duration_ms)`。然后在 `tauri::async_runtime` 上:`transcribe::transcribe(&wav, settings)` 调字节 OpenSpeech → `inject::paste_text()` 写剪贴板 + 合成 ⌘V/Ctrl+V → `HistoryStore::add()` 持久化 → emit `history-updated` 让 popover 刷新

跨线程状态用 `Arc<HistoryStore>` 和 `Arc<SettingsStore>` 传(内部各自包 `parking_lot::Mutex`),通过 `tauri::Manager` 注册,这样 `#[tauri::command]` 处理器能用 `tauri::State<'_, AppState>` 拿到。

### 平台分支(共享 ~77%)

只有两个模块带 `#[cfg(target_os = ...)]`:

- `hotkey/macos.rs` —— CGEventTap 监听 Fn 键;同时通过 `AXIsProcessTrustedWithOptions` 把关辅助功能权限。macOS 的 Fn/Globe 可能会触发系统“切换输入法”,所以这里不能用纯 `ListenOnly` 长期放行;要用可拦截 tap,并对 Fn/Globe 事件 `Drop`,否则会出现第一次能录音、后续 Fn 变成切输入法的问题。
- `hotkey/windows.rs` —— `WH_KEYBOARD_LL` 低层 hook 监听 `VK_RMENU`。静态 sender 用 `OnceLock<UnboundedSender>`(**不要**用 `static mut`)。windows crate 0.58 下 `HMODULE` 要显式包成 `HINSTANCE(hmod.0)`
- `inject::synthesize_paste` —— macOS 用 `CGEventPost(⌘V)`,Windows 用 `SendInput(Ctrl+V)`

其它(cpal 录音、reqwest HTTP、arboard 剪贴板、UI)都是共享的。

### 前端(无框架)

两份静态 HTML,各自有自己的 TS 入口:

- `src/popover.html` + `popover.ts` —— 托盘下拉的 popover,显示录音状态 + 历史列表。监听 `recording-state`、`history-updated`、`popover-opened`;调用 `list_history`、`copy_history_item`、`delete_history_item`、`clear_history`、`open_settings_window`、`hide_popover`、`quit_app`
- `src/settings.html` + `settings.ts` —— API 凭证表单。调用 `get_meta`、`save_settings`、`request_accessibility`

`vite.config.ts` 把两个页面都声明为 input,输出到 `../dist`(也就是 `src/` 的同级,**不是** `src/dist/`),这样 Tauri 的 `frontendDist: "../dist"` 才能找到。

### Popover 定位(macOS 坑)

Retina + 多显示器下,`TrayIconEvent::Click.rect` 给的是物理坐标,而且副屏可能是负值。`lib.rs` 里的 `toggle_popover` 必须取 popover 窗口真实的 `scale_factor()`,显式传进去 —— rect 转换用 `to_physical::<f64>(scale)`,popover 宽度也要换算成物理像素,最后用 `set_position(PhysicalPosition)`。在 Retina 上对 `to_logical()` 传 `scale=1.0` 会把 popover 放到屏外。

### 设置和历史持久化

`SettingsStore`(settings.json)和 `HistoryStore`(history.json,上限 `HISTORY_MAX = 500`)都放在 `dirs::data_dir()/com.vibetalk.dictation/`。这个路径要和 `tauri.conf.json` 的 bundle identifier 保持一致。

`identifier` **不能**以 `.app` 结尾 —— macOS 会把它当成 bundle 扩展名,build 会警告/失败。我们用的是 `com.vibetalk.dictation`。

### macOS bundle 配置

- `app/src-tauri/Info.plist` 会被 tauri-build 自动合并进最终 bundle 的 Info.plist。里面有 `NSMicrophoneUsageDescription`(系统弹麦克风权限框必需)和 `LSUIElement=true`(从 Dock 隐藏,菜单栏 only 应用必备)
- 除了 `LSUIElement`,代码里 `set_accessory_activation_policy` 还显式调了一次 `NSApplicationActivationPolicy::Accessory`
- `app/src-tauri/icons/icon.ico` 必须存在,否则 Windows `tauri build` 会失败("required for generating a Windows Resource file")。从 `icon.png` 用 PIL 重新生成: `Image.save('icon.ico', format='ICO', sizes=[(16,16),...,(256,256)])`

### API 凭证

`config.rs` 里仍然 hard-code 了字节的默认值(APP_ID 等),但 `transcribe::transcribe` 现在接收一个运行时的 `Settings` 快照,所以用户在设置窗口里改的会覆盖这些常量。**不要**再写直接读 `config::APP_ID` 去发请求的代码,统一走 `SettingsStore::get()`。

## 需要保持的约定

- 录音太短的阈值(`wav.len() < 4_000` 字节)是静默丢弃,不要把它当成错误塞进历史
- `transcribe()` 的错误会作为一条 `error: Some(...)`、`text: String::new()` 的 history item 持久化,popover 里会渲染成红色
- 托盘图标:`tray-idle.png` 是黑色 + `icon_as_template(true)`,让 macOS 自动适配菜单栏色调。`tray-recording.png` 是橙色 #ff9500,**不能**作为 template(否则颜色会被吃掉)

---
> Source: [Windy3f3f3f3f/vibetalking](https://github.com/Windy3f3f3f3f/vibetalking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
