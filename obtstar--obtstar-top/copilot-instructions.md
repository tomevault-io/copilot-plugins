## obtstar-top

> > 本文件面向 AI 编码代理，假设读者对本项目一无所知。

# AGENTS.md — ObtStar 网站项目说明

> 本文件面向 AI 编码代理，假设读者对本项目一无所知。

## 项目概述

ObtStar（obtstar.top）是一个**中文（zh-CN）研究报告展示与阅读网站**，主题为「探索 AI 前沿 · 洞察数字未来」。站点发布 AI 相关的深度研究报告（事实核查、数字化转型、提示工程等），提供报告列表、详情页和在线分页阅读器。

项目**没有构建系统、没有包管理器配置**（无 `package.json`、`pyproject.toml` 等），全部由原生技术组成：

- **后端**：原生 Node.js `http` 模块（零依赖），`server.js` + `api/index.js`（CommonJS）。仅用于本地开发或自托管；GitHub Pages 上没有它。
- **前端**：纯静态 HTML + 原生 CSS + 原生 JS（无框架、无打包）。唯一的第三方依赖是通过 CDN 引入的 [AOS](https://michalsnik.github.io/aos/) 滚动动画库。
- **内容管线**：`tools/` 下若干独立的 Python 脚本（依赖 `PyPDF2`），用于把 PDF 研究报告转换为网站的分块数据文件。这些脚本是开发辅助工具，不属于运行时，也不部署。
- **PWA**：`docs/sw.js` Service Worker 提供离线缓存。

注释、文档和提交信息均以**中文**为主，请沿用中文。

## 仓库分层（数据与部署分离）

仓库按「部署产物 / 开发工具」分层：

- **`docs/`** = 部署产物。GitHub Pages 直接从 `main` 分支的 `/docs` 目录发布；`node server.js` 也只服务这个目录。所有运行时文件（页面、样式、脚本、数据）都在里面。
- **仓库根** = 开发工具，不部署：`server.js`（本地开发服务器）、`api/`（本地 API）、`tools/`（PDF 内容管线）。

## 目录结构与模块划分

```
├── docs/                    # 【部署产物】GitHub Pages 发布此目录
│   ├── .nojekyll            # 跳过 GitHub Pages 的 Jekyll 处理
│   ├── *.html               # 页面：index(首页) reports(报告库) report-detail(详情) reader(阅读器) about 404
│   ├── css/styles.css       # 全局样式（约1500行）
│   ├── css/components.css   # 组件样式
│   ├── js/main.js           # 全局动效（3D卡片、视差、光晕），尊重 prefers-reduced-motion
│   ├── js/api-client.js     # 前端 API 封装（fetch /api/*，失败时自动回退纯静态数据）
│   ├── js/theme-manager.js  # 深/浅主题切换（localStorage: obtstar-theme）
│   ├── js/bookmarks.js      # 书签收藏（localStorage: obtstar-bookmarks，上限100条）
│   ├── js/particles.js      # 首页 Canvas 粒子背景
│   ├── sw.js                # Service Worker（静态资源 Cache First，API Network First）
│   ├── data/reports.js      # 【核心数据】CATEGORIES + REPORTS 数组（浏览器直接 <script> 加载）
│   ├── data/reports-content/  # 报告正文：每报告一个目录，含 manifest.json + chunk-N.js
│   └── robots.txt / sitemap.xml / favicon.svg
├── server.js                # 本地开发入口：原生 http 静态服务器（服务 docs/）+ API 路由分发
├── api/index.js             # RESTful API 模块（CommonJS，导出 handleAPIRequest，读取 docs/data/）
├── tools/                   # 内容管线（开发辅助，不部署）
│   ├── generate_chunks.py   # 把提取出的 PDF 文本转成 chunk-N.js + manifest.json
│   ├── validate_chunks.py   # 校验 chunk 文件与 manifest 一致性
│   ├── extract_pdf.py / extract_all_pdfs*.py / read_pdf.py  # PDF 文本提取工具（PyPDF2）
│   └── pdf_content.txt      # 提取出的中间产物
└── .git/                    # 远程：git@github.com:obtstar/obtstar.top.git
```

### 数据流（重要）

报告数据有两条消费路径，**修改数据时需同时考虑两者**：

1. **浏览器直连（GitHub Pages 上唯一可用的路径）**：`reader.html`、`about.html` 用 `<script src="data/reports.js">` 直接加载 `REPORTS`/`CATEGORIES` 全局变量；`reader.html` 再 `fetch data/reports-content/<id>/manifest.json` 和 `chunk-N.js`，通过执行 JS 读取 `window.REPORT_CHUNK_<id下划线化>_<N>`。`index.html`、`reports.html`、`report-detail.html` 走 `js/api-client.js`，API 不可用时自动回退到这条静态路径。
2. **API 路径（仅本地 `node server.js`）**：`api/index.js` 在服务端用正则 + `new Function()` 解析 `docs/data/reports.js` 中的 `REPORTS` 数组（解析失败时回退到内置的 `getFallbackReports()`），按同样方式解析 chunk 文件。

chunk 文件格式约定：

- 文件名为 `chunk-<N>.js`，内容形如 `window.REPORT_CHUNK_ai_fact_check_2026_0 = {...}`，变量名中的报告 id 需把 `-` 替换为 `_`。
- `manifest.json` 含 `id / title / totalPages / totalChunks / chunks[]`。
- 正文 HTML 存于每个 section 的 `content` 字段（模板字符串），阅读器直接注入页面。

## 运行与验证命令

```bash
# 启动本地开发服务器（无 npm，直接用 node；服务 docs/ 目录 + /api/*）
node server.js            # 监听 http://localhost:3001（PORT 环境变量可改）

# 验证 API（仅本地有，GitHub Pages 上不存在 /api/*）
curl http://localhost:3001/api/            # 健康检查在 API 根路径
curl http://localhost:3001/api/reports
curl http://localhost:3001/api/reports/<id>/sections?chunk=0

# 模拟 GitHub Pages 纯静态环境（验证 api-client 静态回退）
python3 -m http.server 8000 -d docs

# 内容管线（需 pip install PyPDF2，建议虚拟环境；在仓库根运行即可，脚本内用相对路径定位 docs/data/）
python3 tools/generate_chunks.py   # 重新生成 chunk 数据
python3 tools/validate_chunks.py   # 校验
```

**没有自动化测试、没有 lint/格式化配置。** 验证改动的方式是：启动 `node server.js`，用浏览器或 curl 检查页面与 API。Python 脚本语法可用 `python3 -m py_compile tools/*.py` 快速检查。

## API 约定（api/index.js，仅本地开发）

- 仅允许 `GET / HEAD / OPTIONS`，其他方法返回 405。
- 路由：`/api/`（健康检查）、`/api/categories`、`/api/reports`（支持 `category/search/tag/sort/page/limit` 查询参数与 RFC 5988 `Link` 分页头）、`/api/reports/:id`、`/api/reports/:id/sections[?chunk=N]`、`/api/reports/:id/manifest`、`/api/tags`、`/api/sources`。
- 遵循 RESTful 规范：RFC 7807 `application/problem+json` 错误格式、ETag/304 条件请求、HATEOAS `links` 字段、CORS `*`。
- 报告 id 校验：`/^[a-z0-9-]+$/` 且长度 ≤ 64。
- 数据有进程内缓存（categories/reports/manifests/chunks），**修改数据文件后需重启服务器**才能生效。
- **查询语义双实现警告**：`docs/js/api-client.js` 的静态回退复刻了 `/api/reports` 的筛选/排序/分页逻辑。修改服务端查询语义时，必须同步修改 api-client 的 `staticGetReports()`。

## 代码风格约定

- **新增报告/分类**：直接编辑 `docs/data/reports.js`，文件头注释里有模板说明；同时在 `api/index.js` 的 `loadCategories()` 和 `getFallbackReports()` 中同步追加（两处硬编码数据需与 `docs/data/reports.js` 保持一致）。
- JS：CommonJS（后端）/ 全局脚本（前端，无模块系统）；JSDoc 风格中文注释；2 空格缩进；前端统一有 `escapeHtml()` 防 XSS 的惯例——**向 DOM 注入动态文本时必须转义**。
- 主题/动效：CSS 变量定义在 `docs/css/styles.css`，深色模式通过 `<html data-theme="dark">` 切换；动画代码需尊重 `prefers-reduced-motion`。
- 提交信息：中文 Conventional Commits，如 `feat: ...`、`refactor(api): ...`（见 `git log`）。

## 部署

- **生产部署 = GitHub Pages**：仓库 Settings → Pages → Source 选 `main` 分支 `/docs` 目录。Pages 只发布 `docs/`，根目录的 server.js / api/ / tools/ 不上线。
- Pages 上无 Node 进程，`/api/*` 不存在；`docs/js/api-client.js` 会在 API 请求失败后自动切换为纯静态模式（直接加载 `data/reports.js` 和 chunk 文件），页面表现一致。
- 本地开发仍用 `node server.js`（静态 + API 都有）。
- **Service Worker 缓存陷阱**：`docs/sw.js` 使用版本化缓存名（`obtstar-v3` 等）。修改静态资源后若用户端不更新，需提升 `CACHE_NAME` 版本号；`reports.html` 中也有带版本查询串引用 JS 的先例（`api-client.js?v=...`）。
- `robots.txt` 禁止爬虫访问 `/api/`、`/data/`、`/js/api-client.js`。

## 安全注意事项

- 前端注入动态内容必须经 `escapeHtml()`；chunk 中的 `content` 是受信任的本地 HTML，直接注入，不要在其中放未清洗的用户输入。
- API 是只读的，无认证；不要为它添加写操作而不加鉴权。
- 服务端用 `new Function()` 解析数据文件——只适用于本仓库自带的受信任数据文件，切勿把该模式用于外部输入。
- `server.js` 已做基本的路径穿越防护（解析后路径必须在 `docs/` 内），但本地服务器不建议直接对公网暴露。

---
> Source: [obtstar/obtstar.top](https://github.com/obtstar/obtstar.top) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
