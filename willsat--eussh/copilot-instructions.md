## eussh

> **eussh** 是一个跨平台 SSH 客户端桌面应用，基于 **Tauri v2** + **Vue 3** 构建，版本 `1.3.7`。

# AGENTS.md

## 项目概览

**eussh** 是一个跨平台 SSH 客户端桌面应用，基于 **Tauri v2** + **Vue 3** 构建，版本 `1.3.7`。

核心功能：多服务器 SSH 连接（Tab 页管理）、终端仿真（xterm.js）、SFTP 文件管理、服务器资源监控面板（CPU/内存/磁盘/网络）、批量命令执行、主机密钥 TOFU 验证、配置加密存储、中英双语支持。

---

## 技术栈

| 层级 | 技术 | 关键依赖 |
|------|------|----------|
| 桌面框架 | Tauri v2 | Rust 系统 WebView |
| 前端 | Vue 3 + Pinia 3 + Vite 8 | xterm.js 6, ECharts 6, Tailwind CSS 4 |
| SSH 后端 | Rust (edition 2021) | `russh` 0.61 (pure Rust SSH), `tokio` |
| 加密存储 | AES-256-GCM + PBKDF2 | `aes-gcm`, `ring` (machine-id 派生密钥) |

---

## 目录结构与职责

```
eussh/
├── src/                          # Vue 前端源码
│   ├── main.js                   # 入口：创建 Vue App + Pinia
│   ├── App.vue                   # 根组件，渲染 AppShell
│   ├── assets/css/main.css       # 全局样式 + Tailwind CSS
│   ├── components/
│   │   ├── common/               # 通用组件：Modal, Toast, RoseSpinner
│   │   ├── connection/           # 连接对话框、主机密钥验证对话框
│   │   ├── filemanager/          # 文件管理器（含右键菜单、面包屑、图标/列表视图）
│   │   ├── layout/               # 布局组件：AppShell, ActivityBar, Sidebar, MainTabBar, StatusBar
│   │   │   └── sidebar/          # 侧边栏视图：ServersView, BatchView, SettingsView
│   │   ├── server/               # 服务器概览面板（CPU/内存/磁盘/地图/IP 列表）
│   │   └── terminal/             # 终端容器（xterm.js 实例管理）
│   ├── composables/              # Vue Composables（复用逻辑）
│   │   ├── useXterm.js           # xterm.js 生命周期管理
│   │   ├── useI18n.js            # 国际化（中/英，支持 {{param}} 模板）
│   │   ├── useTheme.js           # 暗色/亮色主题切换
│   │   ├── useToast.js           # Toast 通知队列
│   │   ├── useLogger.js          # 应用内调试日志（最多 200 条）
│   │   └── useServerData.js      # 服务器动态数据拉取（监控指标 + 静态信息）
│   ├── stores/                   # Pinia 状态管理
│   │   ├── useConnectionStore.js # 连接配置 CRUD
│   │   ├── useServerStore.js     # 运行时服务器状态（Tab、重连、Ping/流量）
│   │   ├── useSettingsStore.js   # 用户偏好设置
│   │   └── useFileManagerStore.js# 文件管理器状态（路径、选中、剪贴板）
│   ├── utils/
│   │   ├── ipc.js                # Tauri IPC 封装（invoke 统一错误处理）
│   │   └── theme.js              # 终端配色方案预设（6 套主题）
│   └── locales/                  # 国际化语言文件
│       ├── en.js
│       └── zh-CN.js
│
├── src-tauri/                    # Rust 后端源码
│   ├── tauri.conf.json           # Tauri 配置（窗口、CSP、打包）
│   ├── Cargo.toml                # Rust 依赖声明
│   ├── capabilities/default.json # Tauri 权限清单
│   └── src/
│       ├── main.rs               # 入口：创建 AppState，注册全部 23 个 Tauri 命令
│       ├── state.rs              # AppState（持有 ConfigStore + SshManager）
│       ├── commands/             # Tauri IPC 命令处理器
│       │   ├── mod.rs
│       │   ├── config.rs         # get_config, save_config, save/delete_connection
│       │   ├── connection.rs     # connect, disconnect, terminal_write/resize, exec, ping, traffic, clipboard, batch_exec, confirm_host_key
│       │   ├── file.rs           # file_list, mkdir, remove, rename, copy, exists, read, write, download_dir, upload_path, chmod
│       │   └── open.rs           # open_url（严格 URL 校验，仅允许 HTTP/HTTPS）
│       ├── ssh/                  # SSH 协议实现
│       │   ├── mod.rs
│       │   ├── manager.rs        # SshManager：会话生命周期、exec 队列、Ping 限速
│       │   ├── session.rs        # SshSession：DNS → TCP 连接 → SSH 握手 → 认证 → PTY → Shell → 读写循环
│       │   └── host_key.rs       # HostKeyVerificationManager：known_hosts 管理、TOFU 验证
│       ├── storage/              # 持久化存储
│       │   ├── mod.rs
│       │   ├── config_store.rs   # ConfigStore：原子读写（tmp 写入 + rename）
│       │   └── encrypt.rs        # AES-256-GCM 加密 + PBKDF2 密钥派生
│       └── models/               # 数据结构
│           ├── mod.rs
│           ├── connection.rs     # ConnectionProfile, AuthMethod（密码 / 私钥）
│           └── config.rs         # AppConfig, AppSettings（字体、配色、监控间隔等）
│
├── public/world.json             # GeoJSON 世界地图数据（Natural Earth 110m）
├── index.html                    # HTML 入口（含主题闪烁防护脚本）
├── vite.config.js                # Vite 配置（端口 5173，`@` 别名指向 src）
├── package.json                  # NPM 配置
└── .github/workflows/            # CI/CD
    ├── build.yml                  # PR/Push 构建（macOS ARM64, Ubuntu x86_64, Windows MSVC）
    └── release.yml               # Tag 触发发布（多平台构建 + GitHub Release）
```

---

## 架构与数据流

### 前后端通信

```
Vue 前端                          Rust 后端
─────────                        ─────────
invoke('command', args)  ──→    Tauri IPC 命令处理器
                              ↓
event.listen('event')  ←──    app_handle.emit('event', data)
```

- **Request/Response**：前端通过 `invoke()` 调用后端命令（`src-tauri/src/commands/` 下的函数），返回 Promise
- **Event Push**：后端主动推送事件到前端（`connection-status`, `terminal-data`, `host-key-verify`, `debug-event`, `ping` 等）

### 前端 IPC 封装（`src/utils/ipc.js`）

```js
import { invoke } from '@/utils/ipc'
const result = await invoke('get_config')
```

所有 Tauri invoke 调用必须通过此封装，统一错误处理和日志。

### 状态管理（Pinia Stores）

| Store | 职责 | 文件 |
|-------|------|------|
| `useConnectionStore` | 连接配置的 CRUD，通过 IPC 读写加密配置 | `src/stores/useConnectionStore.js:7` |
| `useServerStore` | 运行时服务器状态：连接/断开、Tab 管理、重连逻辑、Ping/流量定时器 | `src/stores/useServerStore.js:18` |
| `useSettingsStore` | 用户偏好（主题、字体、终端配色等），从加密配置加载 | `src/stores/useSettingsStore.js` |
| `useFileManagerStore` | 文件管理器状态：当前路径、文件列表、选中项、剪贴板、导航历史 | `src/stores/useFileManagerStore.js` |

### SSH 会话生命周期

```
connect() → DNS 解析 → TCP 连接 (10s 超时)
→ SSH 握手 (15s 超时) → 主机密钥验证 (TOFU)
→ 密码/私钥认证 → PTY 分配 → 启动 Shell
→ stdin 写入通道 + stdout 读取循环 (60s 空闲超时)
→ resize 处理 + exec 命令处理 (信号量限制 5 并发, 30s 超时)
→ disconnect() 清理
```

关键文件：`src-tauri/src/ssh/session.rs:19`（`ClientHandler`），`src-tauri/src/ssh/manager.rs`（`SshManager`）

### Tauri 命令注册模式

所有命令在 `src-tauri/src/main.rs:20-48` 集中注册，按模块拆分到 `commands/` 目录：

```rust
.invoke_handler(tauri::generate_handler![
    config::get_config,
    config::save_config,
    connection::connect,
    file::file_list,
    open::open_url,
    // ... 共 23 个命令
])
```

新增命令时：
1. 在 `src-tauri/src/commands/` 对应的模块文件中添加 `#[tauri::command]` 函数
2. 在 `main.rs` 的 `generate_handler!` 宏中添加命令名称
3. 命令函数签名使用 `app_state: tauri::State<'_, AppState>` 获取全局状态

### 事件系统

后端推送事件到前端（`app_handle.emit("event-name", payload)`）：

| 事件名 | 触发时机 | 载荷 |
|--------|----------|------|
| `connection-status` | SSH 连接状态变化 | `{session_id, status: "connecting"/"connected"/"disconnected"/"error"}` |
| `terminal-data` | SSH 终端输出 | `{session_id, tab_id, data}` |
| `host-key-verify` | 需要用户确认主机密钥 | `{host, fingerprint, session_id}` |
| `debug-event` | 调试日志 | `{session_id, level, message}` |
| `ping` | Ping 延迟测量结果 | `{session_id, latency_ms}` |
| `server-traffic` | 网卡流量数据 | `{session_id, upload_rate, download_rate}` |

前端事件监听模式（以 `AppShell.vue` 为例）：
```js
import { listen } from '@tauri-apps/api/event'
const unlisten = await listen('event-name', (event) => { /* ... */ })
```

### AppState 全局状态

`src-tauri/src/state.rs:5` 定义：

```rust
pub struct AppState {
    pub config_store: ConfigStore,       // 加密配置存储
    pub ssh_manager: Arc<SshManager>,    // SSH 会话管理器（线程安全共享）
}
```

通过 `tauri::Builder::manage()` 注入，命令中通过 `tauri::State<'_, AppState>` 访问。

---

## 开发命令

```bash
# 安装依赖
npm install

# 开发模式（启动 Vite + Tauri）
npx tauri dev

# 仅前端开发（无 Tauri 后端）
npm run dev         # Vite 开发服务器（端口 5173，严格端口）

# 生产构建
npm run tauri build

# 前端构建
npm run build       # vite build
npm run preview     # 预览构建产物

# Rust 检查
cargo check         # 在 src-tauri/ 目录下运行
cargo clippy        # Lint 检查
```

---

## 约定与模式

### 前端约定

1. **路径别名**：`@` 指向 `src/` 目录，Vite 配置见 `vite.config.js:9`
2. **IPC 调用**：统一使用 `src/utils/ipc.js` 的 `invoke()`，禁止直接使用 `@tauri-apps/api/core` 的 invoke
3. **状态管理**：所有跨组件状态使用 Pinia stores，不通过 props 多层传递
4. **日志**：使用 `useLogger` composable（`createLogger('ModuleName')` 创建带模块名的 logger）
5. **国际化**：使用 `useI18n` composable 的 `t('key', {param: 'value'})` 方法，语言文件在 `src/locales/`
6. **主题**：通过 `useSettingsStore` 管理，CSS 变量定义在 `src/assets/css/main.css`，暗色模式通过 `body.dark` class 切换
7. **样式**：使用 Tailwind CSS 4 工具类，组件特定样式写在组件 `<style scoped>` 中
8. **Composables 命名**：以 `use` 开头，Vue 3 Composition API 风格

### Rust 后端约定

1. **命令处理器**：放在 `src-tauri/src/commands/` 下，按功能域分文件（config, connection, file, open）
2. **命令注册**：在 `main.rs` 的 `generate_handler!` 宏中集中注册，不在其他地方分散注册
3. **错误处理**：命令返回 `Result<T, String>`，错误信息以字符串形式传递到前端
4. **异步**：SSH 操作使用 tokio 异步运行时，关键操作有超时保护（connect 10s, handshake 15s, exec 30s, idle 60s）
5. **加密存储**：配置写入遵循「写入临时文件 → rename」原子模式（`src-tauri/src/storage/config_store.rs`）
6. **安全**：URL 打开有严格校验（`src-tauri/src/commands/open.rs`），chmod 仅接受八进制字符串，文件路径做规范化处理
7. **日志**：使用 `app_handle.emit("debug-event", ...)` 将后端日志推送到前端 DebugPanel

### 模型定义

| 结构体 | 文件 | 关键字段 |
|--------|------|----------|
| `AppConfig` | `src-tauri/src/models/config.rs:5` | theme, language, connections, settings |
| `AppSettings` | `src-tauri/src/models/config.rs:28` | font_size, terminal_color_preset, monitor_refresh_secs, etc. |
| `ConnectionProfile` | `src-tauri/src/models/connection.rs` | id (UUID), host, port, username, auth_method |
| `AuthMethod` | `src-tauri/src/models/connection.rs` | Password { password } 或 PrivateKey { key_content, passphrase } |

---

## 配置存储

- **位置**：平台配置目录下的 `eussh/`（如 Linux `~/.config/eussh/`，macOS `~/Library/Application Support/com.eussh.desktop/`）
- **config.enc.json**：加密的 JSON 配置文件（AES-256-GCM, PBKDF2 密钥派生自 machine-id）
- **known_hosts**：明文 SSH 已知主机指纹（格式：`host:port SHA256:xxx`）
- **加密流程**：`machine-id → PBKDF2-HMAC-SHA256 (100K iterations, salt: "eussh-config-salt-v1") → AES-256-GCM (random 12-byte nonce)`

---

## CSP 策略

`src-tauri/tauri.conf.json:25`：

```
default-src 'self'
style-src 'self' 'unsafe-inline'
connect-src 'self' https://api.github.com https://ip-api.com
img-src 'self' blob: data:
```

前端允许的外部网络请求：
- `https://api.github.com` — 版本更新检查
- `https://ip-api.com` — 服务器地理定位

如需添加新的外部 API，必须在此处更新 CSP。

---

## 重要注意事项

1. **无测试套件**：项目当前没有自动化测试（无 `__tests__`、`#[test]`、`vitest`/`jest` 配置），所有测试为手动 QA
2. **重连机制**：`src/stores/useServerStore.js:15-16` 使用 module-level 非响应式 Map/Set 管理重连定时器和取消标记（`_reconnectTimers`, `_reconnectCancel`），采用指数退避（最多 5 次重试）
3. **终端实例**：每个终端 Tab 拥有独立的 xterm.js 实例，通过 `useXterm` composable 管理生命周期，`TerminalContainer.vue` 承载
4. **文件操作**：上传使用 32KB 分块写入（`src-tauri/src/commands/file.rs`），目录下载使用 tar.gz 打包，文件列表优先使用 GNU find，回退到 ls 命令
5. **Ping 限速**：`src-tauri/src/ssh/manager.rs` 中 Ping 操作有 2 秒冷却时间，防止频繁请求
6. **批量执行**：`batch_exec` 命令并行执行，最大 50 个会话
7. **主机密钥验证**：TOFU（Trust-On-First-Use）模式，unknown 主机弹窗确认（60 秒超时），known 主机 key change 弹窗警告
8. **暗色/亮色主题**：在 `index.html` 中有内联脚本防止页面加载时的主题闪烁
9. **平台差异**：macOS 最低版本 10.15，Windows 使用 MSVC，Linux 需要 `libwebkit2gtk-4.1-dev`
10. **版本发布**：CI 在 Tag 推送时触发 `release.yml`，自动构建多平台包并创建 GitHub Release

---
> Source: [WillSat/eussh](https://github.com/WillSat/eussh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
