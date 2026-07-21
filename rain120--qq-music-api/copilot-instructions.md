## qq-music-api

> 本文档用于说明 `qq-music-api` 当前的真实项目架构、`API Explorer` 的模块分层，以及 AI 代理能力在本仓库中的落点与扩展方式。它既面向开发者，也面向需要理解代码结构的大语言模型（LLM）或自动化工作流。

# 🤖 QQ Music API - AI Agents 指南 (AGENTS.md)

本文档用于说明 `qq-music-api` 当前的真实项目架构、`API Explorer` 的模块分层，以及 AI 代理能力在本仓库中的落点与扩展方式。它既面向开发者，也面向需要理解代码结构的大语言模型（LLM）或自动化工作流。

## 🎯 目录

1. [项目概览](#项目概览)
2. [项目架构](#项目架构)
3. [Explorer 架构](#explorer-架构)
4. [AI 代理能力定位](#ai-代理能力定位)
5. [建议的调用与扩展方式](#建议的调用与扩展方式)
6. [开发约束](#开发约束)

---

## 项目概览

`qq-music-api` 是一个基于 `Koa2 + TypeScript` 的服务端项目，核心职责是：

- 对外暴露统一的 QQ 音乐接口。
- 在控制器层接收请求、做参数整理并调用服务层。
- 在服务层请求 QQ 音乐上游能力并返回结构化结果。
- 提供一个内置的本地 `API Explorer` 页面，方便开发环境调试接口。

当前仓库的主体是标准的 Koa 服务和 Explorer 调试工作台。文档中提到的 AI 代理，更适合作为“能力边界”和“扩展方向”来理解，而不是一个已经完整落地的独立运行时框架。

---

## 项目架构

### 1. 服务端主链路

当前请求链路如下：

```text
HTTP Request
  -> Koa App (src/app.ts)
  -> Middlewares (body parser / cookie / cors / response time / logger)
  -> Router (src/routes/router.ts)
  -> Controllers (src/controllers/*)
  -> Services (src/services/*)
  -> QQ Music upstream / data parsing
  -> HTTP Response
```

### 2. 主要目录说明

```text
qq-music-api/
├── src/
│   ├── app.ts                  # Koa 应用入口，注册中间件、Explorer 路由与静态资源
│   ├── config/                 # 服务配置、Explorer 元数据配置
│   ├── controllers/            # 控制器层，处理入参并编排服务调用
│   ├── explorer/               # Explorer 的 contracts/domain/application 逻辑
│   ├── middlewares/            # Koa 中间件
│   ├── routes/                 # 路由注册
│   ├── services/               # QQ 音乐能力封装与数据获取
│   ├── types/                  # 共享类型定义
│   └── util/                   # 公共工具，如 logger、cookie、lyricParse
├── public/
│   └── explorer/               # Explorer 静态页面资源（index.html / app.js / styles.css）
├── tests/                      # Jest + Supertest 测试
├── docs/                       # Docsify 文档与静态截图资源
└── README.md                   # 项目主说明
```

### 3. 关键模块职责

- `src/app.ts`
  - 启动 Koa 服务。
  - 暴露 `/explorer`、`/explorer/index.html`、`/explorer/metadata`。
  - 托管 `public/` 下的静态资源。
  - 在开发环境中配合 `AUTO_OPEN_EXPLORER` 自动打开 Explorer。
- `src/controllers/*`
  - 负责解析查询参数与请求体。
  - 将请求转发到对应 `services`。
  - 封装统一的 HTTP 返回结构。
- `src/services/*`
  - 负责实际的数据拉取、协议拼装与结果加工。
  - 是当前最接近“数据获取代理”的能力落点。
- `src/util/logger.ts`
  - 统一日志出口。
  - Explorer 的服务端调试日志也通过这里输出。
- `tests/*`
  - 覆盖控制器、服务、Explorer 元数据、Explorer 领域逻辑与页面路由。

---

## Explorer 架构

### 1. Explorer 运行方式

`API Explorer` 是仓库内置的本地调试工作台，默认通过以下路径访问：

- 页面入口：`/explorer`
- 静态页面：`/explorer/index.html`
- 元数据接口：`/explorer/metadata`

页面会先拉取 `metadata`，再基于元数据动态生成接口选择器、参数表单、请求配置区和会话日志。

### 2. Explorer 分层

当前 Explorer 已形成清晰的模块边界：

```text
Explorer
├── Metadata Layer
│   └── src/config/apiExplorer.ts
├── Contracts Layer
│   └── src/explorer/contracts/*
├── Domain Layer
│   └── src/explorer/domain/*
├── Application Layer
│   └── src/explorer/application/*
├── View Layer
│   └── public/explorer/*
└── Server Integration
    └── src/app.ts
```

### 3. 各层职责

- `src/config/apiExplorer.ts`
  - 定义接口清单、方法、分类、路径参数、查询参数和 `POST` 示例体。
  - 是 Explorer 动态渲染表单的配置源。
- `src/explorer/contracts/*`
  - 定义 Explorer 状态结构、节点模型、历史记录模型等。
  - 其中状态被拆分为 `resourceState`、`viewState`、`requestState`、`historyState`。
- `src/explorer/domain/*`
  - 负责纯逻辑计算，如构建接口树、筛选可见节点、创建请求默认状态。
  - `selectors.ts` 中实现了搜索关键字与方法筛选的组合过滤。
- `src/explorer/application/*`
  - 负责状态容器与命令式更新逻辑。
  - 当前包含 `ExplorerStore` 与一组 command，用于初始化、切换节点、更新筛选条件等。
- `public/explorer/*`
  - 负责真实页面渲染和浏览器交互。
  - 页面包含方法筛选、接口搜索、动态参数表单、响应预览和 `Logs` 会话日志。
- `src/app.ts`
  - 将 Explorer 与主服务集成。
  - 对 `/explorer` 相关请求输出调试日志，便于排查页面资源或元数据问题。

### 4. Explorer 功能链路

```text
Open /explorer
  -> redirect to /explorer/index.html
  -> fetch /explorer/metadata
  -> build endpoint list and request form
  -> select endpoint / fill params / choose method filter
  -> send HTTP request
  -> render latest response
  -> append request log entry to Logs panel
```

---

## AI 代理能力定位

当前仓库可以从能力上抽象出以下三类代理，它们分别映射到现有代码中的不同层级：

### 1. 🎵 DataFetcherAgent（数据获取代理）

- **当前落点**：`src/services/*`
- **职责**：
  - 请求 QQ 音乐相关接口。
  - 处理服务级数据拼装与返回结构转换。
  - 为控制器提供稳定的数据访问入口。
- **典型场景**：
  - 获取歌曲播放链接
  - 获取歌手、专辑、排行榜、歌单详情
  - 批量查询类接口

### 2. 📝 LyricParserAgent（歌词解析代理）

- **当前落点**：`src/util/lyricParse.ts`、歌词相关 controller/service
- **职责**：
  - 解析歌词文本。
  - 将歌词内容转成前端更易消费的结构。
  - 为歌词接口和后续扩展提供可复用解析能力。
- **典型场景**：
  - `getLyric`
  - 歌词格式化与时间轴处理

### 3. 🔍 SearchOptimizerAgent（搜索优化代理）

- **当前落点**：搜索相关 service/controller 与 Explorer 检索交互
- **职责**：
  - 处理关键字搜索、联想提示、热词等能力。
  - 为接口搜索与调试工作台提供更友好的检索体验。
  - 在 Explorer 中通过搜索框 + 方法筛选快速收敛候选接口。
- **典型场景**：
  - `getSearchByKey`
  - `getSmartbox`
  - Explorer 中的接口快速定位

> 说明：以上“代理”目前主要体现为职责分层与能力抽象，而不是已经存在的 `AgentFactory` 运行时实现。如果后续需要引入独立代理系统，建议在保持现有分层不变的前提下新增 `src/agents/` 目录进行封装。

---

## 建议的调用与扩展方式

### 1. 新增一个数据能力的推荐路径

如果需要新增一个“代理能力”或新的接口编排能力，建议按以下路径实现：

1. 在 `src/services/` 新增或扩展服务逻辑。
2. 在 `src/controllers/` 暴露控制器入口。
3. 在 `src/routes/` 注册路由。
4. 在 `src/config/apiExplorer.ts` 注册 Explorer 元数据，让调试工作台可直接使用。
5. 在 `tests/` 中补齐 controller/service/explorer 相关测试。
6. 更新 `README.md`、`docs/README.md` 与本文件。

### 2. 建议的 LLM 调用上下文

如果由 LLM 或自动化工作流驱动，请优先关注以下上下文：

```json
{
  "target_layer": "service | controller | explorer",
  "intent": "fetch_music_data | parse_lyric | optimize_search | debug_explorer",
  "entry": {
    "http_path": "/explorer/metadata",
    "route": "/getSearchByKey/:key?/:limit?/:page?/:catZhida?"
  },
  "artifacts": [
    "src/config/apiExplorer.ts",
    "src/controllers",
    "src/services",
    "public/explorer"
  ]
}
```

---

## 开发约束

在为本项目增加 AI 代理或扩展 Explorer 时，请遵循以下约束：

1. **保持真实架构一致**：优先复用 `controller -> service -> util` 的现有链路，不要凭空创建与仓库不一致的运行时抽象。
2. **元数据驱动优先**：Explorer 中新增接口调试能力时，应优先扩展 `src/config/apiExplorer.ts`。
3. **状态职责分离**：Explorer 状态需继续遵循 `resourceState`、`viewState`、`requestState`、`historyState` 分层。
4. **统一日志**：服务端日志统一走 `src/util/logger.ts`。
5. **测试与文档同步**：新增能力后，必须同步更新测试与文档。

---
> Source: [Rain120/qq-music-api](https://github.com/Rain120/qq-music-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
