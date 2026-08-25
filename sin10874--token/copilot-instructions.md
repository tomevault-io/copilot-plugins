## token

> OpenClaw & Claude Code 成本监控工具 — 本地运行的 LLM API 用量与费用看板。

# Tokend

OpenClaw & Claude Code 成本监控工具 — 本地运行的 LLM API 用量与费用看板。

## 技术栈

- **前端**: React 18 + Vite + Tailwind CSS 3 + Recharts + React Router 6 + Lucide Icons
- **后端**: Express 4 (TypeScript) + Node.js SQLite (`node:sqlite` DatabaseSync)
- **运行时**: tsx (TypeScript 直接执行)
- **构建**: Vite (前端) / tsx (后端)

## 目录结构

```
src/                    # 前端源码
  components/           # 可复用组件 (Layout, MetricCard, TokenBar)
  pages/                # 页面组件 (Dashboard, Channels, Sessions, Models, Settings, ClaudeCode, SessionDetail)
  lib/                  # 工具库 (api.ts 接口定义与调用, format.ts 格式化函数)
server/                 # 后端源码
  api/routes.ts         # Express API 路由 (所有 GET/POST/PUT 端点)
  db/index.ts           # SQLite 初始化、表结构、种子数据
  ingestion/            # 数据解析层 (scanner 扫描日志文件, parser 解析事件, claude-code-parser/scanner 专用解析)
  index.ts              # Express 入口
data/                   # SQLite 数据库文件 (tokend.db, 自动创建)
dist/client/            # Vite 构建产物
```

## 前端约定

### 颜色变量 (CSS Custom Properties)
- `--bg-base` / `--bg-surface` / `--bg-elevated`: 三层背景色 (深→浅)
- `--text-primary` / `--text-secondary` / `--text-muted`: 三级文字色
- `--border-default` / `--border-subtle`: 边框色
- `--amber` / `--amber-bg` / `--amber-dim`: 主色调 (金色, 用于 Token 指标)
- `--teal`: 辅助色 (用于成本指标、链接)
- `--rose` / `--green` / `--violet` / `--orange`: 语义色

### 字体
- 标题/导航: `Barlow Condensed`, letter-spacing 0.04em
- 数字: `.num` / `.metric-num` class (Barlow 字体, 等宽数字)
- 正文: 系统默认无衬线字体

### 核心组件
- `Layout`: 侧边栏导航 + 主内容区, 支持嵌套分组 (OpenClaw 下有频道/会话子菜单)
- `MetricCard`: 指标卡片, 含 label/value/sub/trend/accent/approx
- `TokenBar`: Token 构成条形图 (输入/输出/缓存读/缓存写)

### 页面结构
每个页面一般包含: 标题区 (PageShell) → 指标卡片行 → 图表区 → 数据表格

## 后端约定

### 数据库 Schema (SQLite)
- `usage_events`: 核心表, 每条 API 调用记录 (token 用量、成本、模型、频道、会话)
- `sessions`: 会话索引表
- `model_prices`: 模型定价表 (USD/百万 Token)
- `ingestion_state`: 增量导入进度
- `source_warnings`: 解析警告
- `claude_code_config`: Claude Code 配额配置 (月预算、计费周期)

### API 路由风格
- RESTful: `GET /api/channels`, `GET /api/channels/:channel`, `GET /api/sessions/:id`
- 查询参数: `?days=N`, `?period=1d|7d|30d`, `?sort=tokens|cost|calls`, `?limit=N&offset=N`
- 响应: 直接 JSON, 无 wrapper (除 POST/PUT 返回 `{ ok: true }`)

### 数据解析层 (ingestion)
- `scanner.ts`: 扫描 `~/.openclaw/sessions/` 下的 JSONL 日志文件
- `parser.ts`: 解析 OpenClaw 格式事件
- `claude-code-scanner.ts` / `claude-code-parser.ts`: 解析 Claude Code 的 `~/.claude/` 目录下的用量数据

## UI 风格

暗色主题, 极简工业风:
- 背景: 深灰系 (#0d0d0d → #1a1a1a → #242424)
- 卡片: `.card` class, 1px 边框, 无阴影
- 间距: Tailwind 标准 (p-3/p-4/p-6, gap-3, space-y-4)
- 字号: 9px (标签) / 10-11px (辅助文字) / 12px (正文) / 标题 text-lg
- 表格: `.data-table` class, 紧凑行高

## 常用命令

```bash
npm run dev          # 同时启动前端 (Vite :4173) 和后端 (Express :3001)
npm run build        # Vite 构建前端到 dist/client/
npm run prod         # 构建 + 生产模式启动
npm run start        # 仅启动生产后端
npm run ingest       # 手动运行数据导入
npx tsc --noEmit     # 类型检查
```

数据库重建: 删除 `data/tokend.db` 后重启服务即可自动重建。

## 开发流程

### 添加新页面
1. 在 `src/pages/` 创建页面组件
2. 在 `src/App.tsx` 添加 Route
3. 在 `src/components/Layout.tsx` 的 NAV 数组添加菜单项

### 添加新 API 端点
1. 在 `server/api/routes.ts` 添加 `router.get/post/put(...)` 路由
2. 在 `src/lib/api.ts` 添加对应的请求函数和 TypeScript 接口
3. 在页面中调用 `api.xxx()` 使用

---
> Source: [Sin10874/token](https://github.com/Sin10874/token) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
