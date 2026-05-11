## web-search-fast

> 基于 **Camoufox + FastAPI** 的高性能 Web 搜索服务，将搜索引擎结果转换为结构化 JSON / Markdown 输出。支持 MCP 协议（Streamable HTTP）供 LLM 客户端直接调用，同时提供 Admin 管理面板。

# Web Search MCP

## 项目概述

基于 **Camoufox + FastAPI** 的高性能 Web 搜索服务，将搜索引擎结果转换为结构化 JSON / Markdown 输出。支持 MCP 协议（Streamable HTTP）供 LLM 客户端直接调用，同时提供 Admin 管理面板。

## 当前版本

**v0.3.6** — 2026-02-23

## 技术栈

| 组件 | 技术 | 用途 |
|------|------|------|
| Web 框架 | FastAPI + Uvicorn + Starlette | HTTP API + MCP 服务 |
| MCP 框架 | FastMCP (mcp>=1.25.0) | MCP 协议实现（stdio / SSE / HTTP） |
| 浏览器引擎 | Camoufox (Playwright) | 反检测 Firefox 浏览器抓取 |
| 异步运行时 | asyncio | 并发调度 |
| HTML 解析 | BeautifulSoup4 / lxml | 页面内容提取 |
| 内容转换 | markdownify | HTML → Markdown |
| 数据库 | SQLite (aiosqlite) | 搜索日志、API Key、IP 封禁 |
| 缓存 | Redis (可选) | IP 封禁缓存 |
| Admin 前端 | React + Vite + Tailwind CSS | 管理面板 SPA |
| 数据校验 | Pydantic v2 | 请求/响应模型 |

## 核心功能

### MCP Tools（3 个）

| Tool | 说明 | 超时 |
|------|------|------|
| `web_search` | 搜索引擎查询，返回 Markdown | 25s |
| `get_page_content` | 获取单个 URL 页面内容 | 20s |
| `list_search_engines` | 列出可用引擎和浏览器池状态 | — |

### 搜索引擎

- **DuckDuckGo**（推荐默认）— HTML-lite 模式，稳定可靠
- **Google** — JS DOM 提取，含首页预热 + 验证码检测 + 同意弹窗处理，被拦截时自动回退
- **Bing** — `global.bing.com` 避免地域重定向，含 URL 解码

### 搜索深度（depth）

| 层级 | 行为 | 说明 |
|------|------|------|
| `depth=1` | SERP 解析 | 默认。提取标题、链接、摘要 |
| `depth=2` | SERP + 正文抓取 | 进入每个结果链接，提取页面正文 |
| `depth=3` | SERP + 正文 + 外链 | 继续抓取正文中的外部链接内容 |

### Camoufox 指纹浏览器

内置功能（默认启用）：GeoIP 伪装、人性化操作、图片阻止、Locale 匹配

可配置功能（环境变量）：

| 环境变量 | 说明 |
|----------|------|
| `BROWSER_POOL_SIZE` | 浏览器并发数 |
| `BROWSER_PROXY` | 代理服务器（socks5/http） |
| `BROWSER_OS` | 目标 OS 指纹（windows/macos/linux） |
| `BROWSER_FONTS` | 自定义字体列表 |
| `BROWSER_BLOCK_WEBGL` | 阻止 WebGL 指纹 |
| `BROWSER_ADDONS` | Firefox 插件路径 |

### Admin 管理面板

- Dashboard 统计、搜索历史、IP 监控、API Key 管理
- 三层中间件：SearchLogMiddleware → APIKeyAuthMiddleware → IPBanMiddleware
- SQLite 持久化 + Redis 可选缓存

## 项目结构

```
web-search-mcp/
├── CLAUDE.md
├── pyproject.toml
├── README.md
├── Dockerfile / Dockerfile.dev
├── docker-compose.yml
├── scripts/
│   └── mcp-server.sh          # MCP 注册/更新脚本
├── src/
│   ├── __init__.py
│   ├── main.py                 # FastAPI 入口
│   ├── mcp_server.py           # MCP 服务入口（FastMCP + 中间件 + Admin）
│   ├── config.py               # 配置管理（BrowserConfig / AppConfig / AdminConfig）
│   ├── api/
│   │   ├── routes.py           # HTTP API 路由
│   │   └── schemas.py          # Pydantic 请求/响应模型
│   ├── core/
│   │   └── search.py           # 框架无关的搜索逻辑（MCP + HTTP 共用）
│   ├── engine/
│   │   ├── base.py             # 搜索引擎抽象基类 + 诊断日志
│   │   ├── google.py           # Google（JS DOM 提取 + 预热 + 验证码）
│   │   ├── bing.py             # Bing（global.bing.com + URL 解码）
│   │   └── duckduckgo.py       # DuckDuckGo（HTML-lite 模式）
│   ├── scraper/
│   │   ├── browser.py          # BrowserPool（tab-per-search 隔离）
│   │   ├── parser.py           # HTML 内容解析
│   │   └── depth.py            # 多层深度抓取（domcontentloaded + 时间预算）
│   ├── formatter/
│   │   ├── json_fmt.py         # JSON 格式化
│   │   └── markdown_fmt.py     # Markdown 格式化
│   ├── admin/
│   │   ├── database.py         # SQLite 初始化 + 迁移
│   │   ├── models.py           # Admin 数据模型
│   │   ├── repository.py       # 数据访问层（API Key / IP Ban / 搜索日志）
│   │   ├── routes.py           # Admin REST API 路由
│   │   └── static/             # Admin SPA 构建产物
│   └── middleware/
│       ├── api_key_auth.py     # Bearer Token 认证（DB + 环境变量）
│       ├── ip_ban.py           # IP 封禁中间件
│       └── search_log.py       # 搜索日志 ASGI 中间件
├── admin-ui/                   # Admin 前端源码（React + Vite + Tailwind）
├── tests/
│   ├── test_api.py
│   ├── test_engine.py
│   ├── test_scraper.py
│   ├── test_formatter.py
│   ├── test_admin_api.py
│   ├── test_browser_pool.py
│   └── test_middleware.py
└── docs/
    ├── CHANGELOG.md
    └── tasks/
```

## 运行方式

### Docker（推荐）

```bash
docker compose up -d
# MCP 端点: http://127.0.0.1:8897/mcp
# Admin 面板: http://127.0.0.1:8897/admin
# 健康检查: http://127.0.0.1:8897/health
```

### 本地开发

```bash
pip install -e ".[dev]"
python -m src.mcp_server --transport http --host 127.0.0.1 --port 8897
```

### Claude Code MCP 注册

已注册在 `~/.claude.json` 用户级 `mcpServers`：
- 名称: `web-search-fast`
- 传输: HTTP (Streamable HTTP)
- 端点: `http://127.0.0.1:8897/mcp`
- 认证: Bearer Token（API Key 存储在 Admin 数据库中）

快捷更新命令：
```bash
./scripts/mcp-server.sh update        # 本地模式
./scripts/mcp-server.sh docker-update  # Docker 模式
```

## 并发策略

- **BrowserPool**: 单 Camoufox 实例 + Semaphore 并发控制
- **Tab 隔离**: 每次搜索 `new_page()` → 执行 → `page.close()`，无 cookie/session 污染
- **页面级并发**: depth>=2 时多个结果页面并行抓取
- **引擎回退**: Google → DuckDuckGo → Bing

## 超时策略

| 层级 | 超时 | 说明 |
|------|------|------|
| MCP `web_search` | 25s | `asyncio.wait_for` 强制执行 |
| MCP `get_page_content` | 20s | 单页面获取 |
| 导航超时 | 10s | `page.goto` 单次导航 |
| 导航重试 | 1 次 | 最坏情况 20s |
| 深度抓取 | 剩余时间预算 | 非固定超时 |
| `domcontentloaded` | 12s 上限 | 替代 `load`（30s） |

## 开发规范

### 命令

```bash
# 安装依赖
pip install -e ".[dev]"

# 运行测试（79 个）
pytest tests/ -v

# 类型检查
mypy src/

# 代码格式化
ruff check src/ --fix
ruff format src/
```

### 代码风格

- Python 3.11+
- 异步优先：所有 I/O 操作使用 async/await
- 类型注解：所有公开函数必须有类型注解
- Pydantic v2 用于数据校验
- 遵循 PEP 8，使用 ruff 格式化

### 测试要求

- 每个功能模块必须有对应的测试文件
- 使用 pytest + pytest-asyncio
- 关键路径需要集成测试

### Git 规范

- Commit message 使用 Conventional Commits 格式
- 不包含 AI 辅助相关字样
- 不包含 anthropic 相关字样

### 文档管理

- 变更日志：`docs/CHANGELOG.md`
- 任务记录：`docs/tasks/【任务名】.md`
- `docs/` 目录在 `.gitignore` 中忽略

---
> Source: [uk0/web-search-fast](https://github.com/uk0/web-search-fast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
