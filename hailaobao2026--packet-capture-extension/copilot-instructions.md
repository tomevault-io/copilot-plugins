## packet-capture-extension

> Chrome Extension（Manifest V3）HTTP 抓包归档工具。当前版本：`v0.0.2`。基于 `chrome.debugger` (CDP) 实现完整的请求/响应捕获，支持会话管理、过滤、多格式导出、AI 解读和单条请求 AI 分析。

# Packet Capture Extension - CLAUDE.md

## 项目概述

Chrome Extension（Manifest V3）HTTP 抓包归档工具。当前版本：`v0.0.2`。基于 `chrome.debugger` (CDP) 实现完整的请求/响应捕获，支持会话管理、过滤、多格式导出、AI 解读和单条请求 AI 分析。

AI 解读功能会把当前抓包会话或用户选中的单条请求中的 URL、请求头/体、响应头/体、状态码、耗时和失败信息整理成摘要，发送到用户配置的大模型接口，返回业务链路、关键接口、参数/返回字段解释、异常风险和排查建议。当前支持 OpenAI 兼容接口和 Anthropic Claude 接口，支持独立配置模型调用 Base URL 与模型列表 URL。

## 技术栈

- Chrome Extension Manifest V3
- Service Worker（后台脚本）
- IndexedDB（持久化存储）
- Chrome DevTools Protocol (CDP) 1.3
- Fetch API（Service Worker 侧调用模型接口）
- 原生 HTML/CSS/JS，无框架依赖

## 项目结构

```
├── manifest.json       # MV3 清单，声明权限和入口
├── background.js       # Service Worker，CDP 抓包核心逻辑
├── popup.html          # Popup 页面结构（实时抓包 / 历史归档 / AI 解读）
├── popup.css           # 暗色主题样式、AI 面板样式、滚动输出区
├── popup.js            # Popup 交互逻辑、消息通信、AI 配置保存、模型下拉选择
├── CLAUDE.md           # 项目维护说明
└── README.md           # 用户文档
```

## 核心架构

### 数据流

```
Popup (UI) ──sendMessage──> Service Worker ──CDP──> Chrome Tab
                               │
                               ├──fetch──> OpenAI/Anthropic/兼容模型网关
                               │
                               ▼
                          IndexedDB
                     ┌─────────┴─────────┐
                  sessions          requests
               (会话元数据)      (请求/响应详情)
```

### 关键 API

| 组件 | API | 用途 |
|------|-----|------|
| 抓包核心 | `chrome.debugger` | 附加到 Tab，通过 CDP 捕获网络事件 |
| 状态持久化 | `chrome.storage.local` | 保存抓包状态、会话 ID（SW 重启恢复） |
| 文件下载 | `chrome.downloads` | 导出 HAR/JSON 文件 |
| 后台保活 | `chrome.alarms` | Service Worker 30s 唤醒一次 |
| 数据存储 | IndexedDB | 请求和会话的持久化存储 |
| AI 调用 | `fetch` | 调用 OpenAI 兼容或 Anthropic 模型接口 |

### CDP 事件处理

`background.js` 监听 4 个核心 CDP Network 事件：

1. `Network.requestWillBeSent` → 记录请求 URL、方法、Headers、Body
2. `Network.responseReceived` → 记录响应状态、Headers、Content-Type
3. `Network.loadingFinished` → 获取响应 Body（`Network.getResponseBody`），持久化
4. `Network.loadingFailed` → 标记失败请求，持久化

### 消息协议

Popup 与 Service Worker 通过 `chrome.runtime.sendMessage` 通信：

| action | 方向 | 说明 |
|--------|------|------|
| `startCapture` | popup → sw | 开始抓包，传入 tabId |
| `stopCapture` | popup → sw | 停止抓包 |
| `getStatus` | popup → sw | 获取当前状态 |
| `getRequests` | popup → sw | 查询请求列表（支持过滤） |
| `getRequestDetail` | popup → sw | 获取单个请求详情 |
| `getSessions` | popup → sw | 获取所有历史会话 |
| `exportHAR` | popup → sw | 构建 HAR 数据并返回给 popup 下载 |
| `exportJSON` | popup → sw | 构建原始 JSON 数据并返回给 popup 下载 |
| `listAIModels` | popup → sw | 根据 AI 配置获取模型列表，支持独立模型列表 URL |
| `analyzeCapture` | popup → sw | 汇总当前会话抓包数据并调用模型分析 |
| `analyzeRequest` | popup → sw | 对用户选中的单条请求/响应调用模型分析 |
| `clearRequests` | popup → sw | 清空请求数据 |
| `deleteSession` | popup → sw | 删除会话及其请求 |

### AI 解读链路

AI 相关逻辑集中在：

- `popup.html`：`AI解读` 标签页、顶部 `AI分析` 按钮、模型配置表单、模型下拉框、自定义模型输入
- `popup.js`：读取/保存 AI 配置、获取模型列表、触发会话分析、触发单条请求分析、展示/滚动查看/复制结果
- `background.js`：整理会话或单条请求抓包快照、脱敏敏感字段、调用模型接口

#### 支持的模型接口

| provider | 默认 Base URL | 模型列表 | 分析接口 | 鉴权 |
|----------|---------------|----------|----------|------|
| `openai` | `https://api.openai.com` | `GET /v1/models`，可单独配置模型列表 URL | `POST /v1/chat/completions` | `Authorization: Bearer <apiKey>` |
| `anthropic` | `https://api.anthropic.com` | `GET /v1/models`，可单独配置模型列表 URL | `POST /v1/messages` | `x-api-key: <apiKey>` |

OpenAI provider 兼容 NewAPI、OneAPI、自建代理等标准 OpenAI 格式网关。`baseUrl` 会自动去除末尾 `/` 和 `/v1`，调用时再拼接 `/v1/...`。`modelsBaseUrl` 可填完整 `/v1/models` 地址，也可填 OpenAI/Anthropic 兼容 Base URL；留空时使用 `baseUrl` 获取模型列表。

#### AI 配置存储

配置保存在 `chrome.storage.local` 的 `aiConfig`：

```js
{
  provider: 'openai' | 'anthropic',
  baseUrl: string,
  modelsBaseUrl: string,
  apiKey: string,
  model: string,
  includeBodies: boolean,
  redactSensitive: boolean,
  maxRequests: number
}
```

#### 发送给模型的数据

`analyzeCapture` 会读取当前会话请求，默认只发送最后 40 条，避免上下文过大。`analyzeRequest` 只读取用户选中的单条请求。每条请求包含：

- URL、Method、Resource Type、开始时间、耗时
- Request Headers / Body
- Response Status、Headers、Content-Type、Size、Body
- 失败请求的 `errorText`
- 实时未完成请求会标记为 `PENDING`

单条请求入口在请求详情面板的 `AI分析此请求` 按钮，适合解释 GET/POST 接口的请求参数、响应字段、错误状态、鉴权/CORS/风控风险。

#### 脱敏策略

默认开启 `redactSensitive`，会对 Header、URL query、JSON/Form Body 中的敏感键进行遮蔽。匹配关键词包括：

`authorization`、`cookie`、`set-cookie`、`token`、`secret`、`password`、`api-key`、`session`、`jwt`、`csrf`、`xsrf`、`credential`、`signature`、`sign`、`auth` 等。

### IndexedDB Schema

**requests store**：`{ id (auto), sessionId, url, method, status, requestHeaders, responseHeaders, requestBody, responseBody, contentType, size, duration, failed, errorText, timestamp, ... }`

**sessions store**：`{ id, tabId, tabUrl, startTime, endTime, requestCount, name }`

## 开发约束

- 响应体超过 500KB 自动截断，防止 IndexedDB 溢出
- AI 分析发送前会对单个 Body 再截断（默认 6000 字符），避免模型上下文过大
- AI 分析默认仅发送当前会话最后 40 条请求，可在 UI 中调整
- 单条请求 AI 分析只发送选中的一条请求/响应，仍遵守 Body 截断和脱敏策略
- AI 配置中的 API Key 当前保存在 `chrome.storage.local`，适合本地自用；不要在不可信环境共享浏览器配置
- 模型调用 `baseUrl` 与模型列表 `modelsBaseUrl` 分离，适配网关调用地址和标准模型列表地址不同的场景
- Service Worker 可被浏览器随时挂起，抓包状态通过 `chrome.storage.local` 恢复
- 抓包期间浏览器顶部出现黄色调试提示条（CDP 机制，无法去除）
- 抓包期间不要按 F12 打开 DevTools，会导致调试器冲突
- 导出由 popup 端调用 `chrome.downloads.download` 完成（SW 端 `URL.createObjectURL` 不可用）

## 构建与安装

无需构建步骤。直接加载到 Chrome：

1. `chrome://extensions/` → 开发者模式 → 加载已解压的扩展程序
2. 选择本项目根目录
3. 点击扩展图标使用

## 修改注意事项

- 修改 `background.js` 后需在 `chrome://extensions/` 点击刷新按钮
- 修改 `popup.html/css/js` 后关闭再打开 popup 即可生效
- `manifest.json` 修改权限后需要重新加载扩展

## 版本历史

### v0.0.2

- 插件版本升级到 `0.0.2`
- 新增单条请求 AI 分析：选中请求后可单独解释请求参数、响应字段和异常风险
- 新增模型列表 URL 配置：模型调用地址和模型列表地址可分离
- 模型选择从文本输入升级为下拉列表，保留自定义模型输入
- 默认 Popup 窗口调整为 460x680，宽度减半、高度保持不变
- 更新 README 和本维护文档

### v0.0.1

- 新增 AI 解读标签页
- 支持 OpenAI 兼容接口和 Anthropic Claude 接口
- 支持自定义 Base URL、请求/响应 Body 开关、敏感字段脱敏
- 优化 Service Worker 状态恢复

---
> Source: [hailaobao2026/packet-capture-extension](https://github.com/hailaobao2026/packet-capture-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
