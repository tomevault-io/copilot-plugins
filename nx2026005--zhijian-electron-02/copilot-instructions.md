## zhijian-electron-02

> Electron + Vue 3 内容创作桌面平台，通过 MCP Server（子进程 HTTP 模式）接入 27 个图像处理/文件操作工具。

# nowCreate-electron

Electron + Vue 3 内容创作桌面平台，通过 MCP Server（子进程 HTTP 模式）接入 27 个图像处理/文件操作工具。

---

## ⚠️ 关键陷阱（新会话必读）

### 1. 系统环境变量 ELECTRON_RUN_AS_NODE=1

该机器上将此变量设为了**系统级环境变量**，值 `=1`。Electron 判断该变量**是否存在**（不判断值），存在即强制以 Node.js 模式运行，后果：

- `require('electron')` / `import ... from 'electron'` 返回 `node_modules/electron/index.js` 导出的**字符串路径**（指向 electron.exe），而非 Electron API 对象
- `app` `BrowserWindow` `ipcMain` 等全部为 `undefined`
- Electron 窗口无法创建，所有 Electron API 调用崩溃

**正确启动方式**：

```bash
# ✅ 通过项目封装脚本启动（推荐）
yarn dev

# ✅ 手动 unset 后直接启动
unset ELECTRON_RUN_AS_NODE && npx electron-vite dev

# ❌ 不要直接运行 electron.exe（会继承系统环境变量）
npx electron .                            # 报错：electron.app.whenReady is not a function
./node_modules/.bin/electron .            # 同上
```

`scripts/start-dev.js` 通过 `delete env.ELECTRON_RUN_AS_NODE` 清除该变量后 spawn electron-vite，**`yarn dev` 已自动处理此问题**。若需直接跑 electron 命令调试，务必先 `unset ELECTRON_RUN_AS_NODE`。

**排查标志**：报错信息中出现以下任一即为此问题 ——
- `TypeError: Cannot read properties of undefined (reading 'exports')`（ESM 模式）
- `TypeError: Cannot read properties of undefined (reading 'whenReady')`（CJS 模式）
- `require('electron')` 返回字符串而非对象

### 2. 不要手动改 package.json 的 "type" 字段

项目根 `package.json` 设 `"type": "module"`。删除该字段会导致 electron-vite 将主进程打包为 CJS（`require` 语法），与 Electron 内嵌的模块加载器产生新的冲突。**该字段为构建系统所用，勿动。**

### 3. MCP Server 路径硬编码

`src/main/mcp-launcher.js` 中 `MCP_SERVER_DIR` 硬编码为 `D:\\leh_project\\sundries\\mcp-server`。生产打包前需改为动态路径（如 `path.join(app.getAppPath(), 'mcp-server')` 并将 MCP Server 目录复制到打包产物中）。

### 4. Node 版本差异

| 环境 | Node 版本 |
|------|-----------|
| 系统 | v18.20.8 |
| Electron 内嵌 | v20.18.0 |
| MCP Server 要求 | >= 18.0.0 |

注意 `import.meta.dirname` 在 Node 18 不可用（21.2+ 才支持），主进程/脚本中使用 `dirname(fileURLToPath(import.meta.url))` 替代。

---

## 目录结构

```
nowCreate-electron/
├── package.json                     # 依赖 + 脚本
├── electron.vite.config.mjs         # electron-vite 构建配置（main/preload/renderer）
├── mcp.json                         # 外部 AI 客户端 MCP 配置模板
│
├── scripts/
│   └── start-dev.js                 # 开发启动封装（清理 ELECTRON_RUN_AS_NODE）
│
├── src/main/                        # Electron 主进程
│   ├── index.js                     # 入口 → app.whenReady 流程
│   ├── mcp-client.js                # MCP HTTP 客户端（session 管理）
│   ├── mcp-launcher.js              # MCP Server 子进程生命周期
│   └── mcp-ipc.js                  # IPC 桥接（ipcMain.handle → HTTP 转发）
│
├── src/preload/
│   └── index.js                     # contextBridge → window.mcp
│
├── src/renderer/                    # Vue 3 渲染进程
│   ├── index.html                   # Vite 入口 HTML
│   └── src/
│       ├── main.js                  # Vue 根实例（Element Plus + Pinia + Router）
│       ├── App.vue                  # el-config-provider 包裹
│       ├── router/index.js          # Vue Router（hash 模式）
│       ├── stores/index.js          # Pinia（useProjectStore / useUiStore）
│       ├── composables/
│       │   └── useMcpTools.js       # window.mcp 调用封装
│       ├── components/
│       │   ├── layout/index.vue     # 主布局（顶栏 + 侧面板 + 画布 + AI 聊天）
│       │   ├── canvas/index.vue     # 画布区域（占位）
│       │   └── chat/index.vue       # AI 聊天面板
│       └── views/
│           ├── home/index.vue       # 首页
│           └── system/notFind.vue   # 404
│
└── resources/                       # 打包资源（icon 等）
```

---

## 启动命令

```bash
yarn dev            # 开发模式（electron-vite dev + HMR）
yarn build          # 生产构建
yarn preview        # 预览构建产物
```

首次安装依赖（如缺少 node_modules 或 yarn.lock）：

```bash
ELECTRON_MIRROR=https://npmmirror.com/mirrors/electron/ yarn install
```

---

## 架构

```
┌─────────────────────────────────────────────────────────────────┐
│  Electron App                                                    │
│                                                                  │
│  ┌──────────────────┐     IPC      ┌──────────────────────────┐  │
│  │  渲染进程          │◄──────────►│  主进程                    │  │
│  │  Vue 3 + Element  │  window.mcp │  ipcMain.handle('mcp:*')  │  │
│  │  ChatPanel.vue    │             │  mcp-ipc.js               │  │
│  │  Canvas.vue       │             │         │                  │  │
│  │  LayerPanel.vue   │             │    HTTP POST /mcp          │  │
│  └──────────────────┘             │         ▼                  │  │
│                                    │  ┌──────────────────┐     │  │
│                                    │  │ MCP Server 子进程  │     │  │
│                                    │  │ localhost:9100    │     │  │
│                                    │  │ (HTTP 模式)       │     │  │
│                                    │  │        │          │     │  │
│                                    │  │   execFile        │     │  │
│                                    │  │        ▼          │     │  │
│                                    │  │  nxImage.exe      │     │  │
│                                    │  │  (27 tools)       │     │  │
│                                    │  └──────────────────┘     │  │
│                                    └──────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  外部 AI 接入（可选）                                       │    │
│  │  Claude Code → stdio → node src/index.js（直连）           │    │
│  │  配置：~/.claude/mcp.json                                   │    │
│  └──────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

| 链路 | 路径 | 用途 |
|------|------|------|
| 内部 AI 聊天 | 渲染进程 → IPC → 主进程 → HTTP → MCP Server | App 内置 AI 面板调用工具 |
| 外部 AI | Claude Code → stdio → MCP Server（直连） | 外部 AI 操控工具 |

---

## 主进程模块详解

### 启动流程

`src/main/index.js` 中：

```
app.whenReady()
  → registerMcpIpc()        // 注册 IPC handler（不依赖 MCP 就绪）
  → startMcpServer()        // spawn MCP Server 子进程 + 健康检查 + initialize session
      ├─ 成功 → console.log 就绪
      └─ 失败 → console.error（不阻塞窗口创建，UI 显示离线状态）
  → createWindow()          // 创建 BrowserWindow
```

### mcp-client.js — MCP HTTP 客户端

- 维护单一 session（`sessionId` + `nextId`）
- 按 Streamable HTTP 协议：首次 POST 无 sessionId → 服务器返回 `mcp-session-id` header → 后续请求附加该 header
- 导出：`initialize()` `listTools()` `callTool(name, args)` `resetSession()`
- 所有函数使用顶层 `fetch`（Electron 28+ 主进程内置）

### mcp-launcher.js — MCP Server 子进程管理

```javascript
spawn('node', ['src/index.js'], {
  cwd: MCP_SERVER_DIR,           // ← 硬编码 D:\leh_project\sundries\mcp-server
  env: {
    MCP_TRANSPORT: 'http',
    MCP_PORT: '9100',
    NX_IMAGE_PATH: join(MCP_SERVER_DIR, 'nxImage.exe')
  }
})
```

健康检查：每 200ms GET `/health`，最多 75 次（15 秒），通过后自动调 `initialize()` 建立 session。

### mcp-ipc.js — IPC 注册

```javascript
ipcMain.handle('mcp:listTools', ...)   // → mcpClient.listTools()
ipcMain.handle('mcp:callTool', ...)    // → mcpClient.callTool(name, args)
```

返回统一格式 `{success: true, data: ...}` 或 `{success: false, error: ...}`。

### preload — 渲染进程暴露

```javascript
contextBridge.exposeInMainWorld('mcp', {
  listTools: () => ipcRenderer.invoke('mcp:listTools'),
  callTool: (name, args) => ipcRenderer.invoke('mcp:callTool', name, args)
});
```

---

## 渲染进程详解

### 依赖

| 包 | 版本 | 用途 |
|----|------|------|
| vue | ^3.4.21 | 框架 |
| element-plus | ^2.7.0 | UI 组件库 |
| @element-plus/icons-vue | ^2.3.1 | 图标 |
| pinia | ^2.1.7 | 状态管理 |
| vue-router | ^4.3.0 | 路由（createWebHashHistory） |
| sass | ~1.77.0 | SCSS 预处理（固定版本，原因见"已知问题"） |

### electron-vite 构建配置（`electron.vite.config.mjs`）

三目标构建：

| 目标 | 入口 | 输出 | 插件 |
|------|------|------|------|
| main | `src/main/index.js` | `out/main/index.js` | `externalizeDepsPlugin()` |
| preload | `src/preload/index.js` | `out/preload/index.mjs` | `externalizeDepsPlugin()` |
| renderer | `src/renderer/index.html` | `out/renderer/` | `@vitejs/plugin-vue` |

SCSS 全局注入：`additionalData: '@use "@/assets/css/variate.scss" as *;'`

**注意**：electron-vite 2.x 要求 rollupOptions.input 的 key 为 `index`，否则输出文件名与 CLI 期望不匹配（如 `main.js` 而非 `index.js`）。

### 组件通信

| 方向 | 方式 |
|------|------|
| 父 → 子 | props |
| 子 → 父 | emit |
| 跨组件 | Pinia stores |
| 渲染 ↔ 主进程 | IPC（window.mcp / ipcMain.handle） |

### useMcpTools() composable

```javascript
const {tools, loading, error, loadTools, callTool} = useMcpTools();
// loadTools() → 获取全部 27 个工具 → 更新 uiStore.mcpOnline + toolCount
// callTool('nx_filter', {input, output, filter:'sepia'}) → 调用 MCP 工具
```

---


## 外部 AI 接入

MCP Server 原生支持 Stdio 模式，外部 AI 客户端可直接连接，无需 bridge。

配置 `~/.claude/mcp.json` 或对应客户端的 MCP 配置：

```json
{
  "mcpServers": {
    "nowCreate": {
      "command": "node",
      "args": ["D:\leh_project\sundries\mcp-server\src\index.js"],
      "env": {
        "NX_API_KEY": "your-key"
      }
    }
  }
}
```

> 前提：MCP Server 已安装依赖（`npm install`），`NX_API_KEY` 已配置。获取 Key 请联系微信 zhjian_2026。

---

## 已知问题与解决方案

| 问题 | 现象 | 解决方案 |
|------|------|----------|
| `ELECTRON_RUN_AS_NODE=1` | `require('electron')` 返回路径字符串，app/BrowserWindow 为 undefined | `yarn dev`（已自动处理）或手动 `unset` |
| sass 版本不兼容 | `error sass: The engine "node" is incompatible` | sass 固定 `~1.77.0`（1.80+ 要求 Node >= 20.19） |
| electron 下载失败 | `RequestError: read ECONNRESET` | `ELECTRON_MIRROR=https://npmmirror.com/mirrors/electron/` |
| MCP Server 启动超时 | App 启动但工具不可用 | 首次加载 SDK 慢，已延长到 15s；若仍超时，MCP Server 依赖（express + MCP SDK）可能未安装 |
| windowsHide 警告 | spawn 子进程时控制台闪现 | mcp-launcher 中设 `windowsHide: true` 可消除 |
| import.meta.dirname 报错 | Node 18 不支持 | 全部使用 `dirname(fileURLToPath(import.meta.url))` |
| SCSS @import 弃用警告 | Dart Sass 警告 | 已改用 `@use '...' as *;` |
| 渲染进程 dev 端口冲突 | 5173 被占用 | electron-vite 自动切换端口，注意 `ELECTRON_RENDERER_URL` 随之变化 |

---

## MCP Server 依赖速查

| 属性 | 值 |
|------|-----|
| 位置 | `D:\leh_project\sundries\mcp-server`（独立仓库） |
| 包名/版本 | `nx-mcp-server` v1.0.5 |
| SDK | `@modelcontextprotocol/sdk` ^1.0.0 |
| 依赖 | `express` ^4.19.0（仅 HTTP 模式） |
| 引擎 | `nxImage.exe`（C / MSYS2 MinGW-w64） |
| 工具 | 15 图像（convert/compress/resize/crop/rotate/flip/adjust/filter/watermark/montage/info/diff/steg/pipeline/batch）+ 12 文件 |
| HTTP 端点 | `POST /mcp` `GET /mcp` `DELETE /mcp` `GET /health` |
| 传输选择 | `MCP_TRANSPORT=stdio`（默认）/ `MCP_TRANSPORT=http` |

---

## 调试提示

```bash
# 验证 MCP Server 是否独立可用
cd D:\leh_project\sundries\mcp-server
npm run start:http        # 启动 HTTP 模式，curl http://localhost:3000/health 验证

# 验证 Electron 窗口能否创建（最小测试）
unset ELECTRON_RUN_AS_NODE
node -e "require('electron')"   # 这条在系统 Node 下执行只会返回路径字符串，正常
npx electron -e "const {app}=require('electron'); console.log(typeof app)"
# ↑ 若输出 'object' → Electron 正常；若输出 'string' → 仍有 ELECTRON_RUN_AS_NODE 问题

# 清除构建缓存
rm -rf out/ node_modules/.vite

# 查看渲染进程控制台
# Dev 模式下 window 会自动打开 DevTools，也可 mainWindow.webContents.openDevTools()
```

---
> Source: [nx2026005/zhijian_electron_02](https://github.com/nx2026005/zhijian_electron_02) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
