## folio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 产品定位

**Collect** — 类似 Reeder 的现代信息阅读工具，专注于优雅的 RSS 阅读体验。

### 核心价值
- **沉浸式阅读**：自动提取文章正文，提供类似 Safari 阅读模式的纯净体验
- **高效管理**：文件夹分类、智能筛选、滑动操作，快速处理信息流
- **双语阅读**：内置谷歌翻译，支持原文/译文/双语对照三种模式
- **跨端同步**：网页端 + iOS 原生端，无缝切换阅读进度

### 目标用户
- 信息密度高的深度阅读者
- 需要追踪多个博客/新闻源的内容创作者
- 需要双语阅读的外语学习者
- 注重阅读体验的 RSS 老用户（从 Reeder/Inoreader 迁移）

## 技术架构

采用 pnpm monorepo 管理，三端统一：

| 子包 | 技术栈 | 部署目标 | 说明 |
|------|--------|----------|------|
| `api/` | Hono + Cloudflare Workers + D1 + Better Auth | Cloudflare Workers | 统一 REST API |
| `frontend/` | Astro 6 + React Islands + Tailwind CSS v4 + shadcn/ui | Cloudflare Pages | 网页端（桌面优先） |
| `mobile/` | Flutter 3 + Riverpod + go_router | iOS / Android | 移动端（iOS 优先体验） |

共享包：
- `packages/types/` — 共享 TypeScript 类型和 Zod schema
- `packages/config/` — 共享 ESLint 和 TypeScript 配置

## 核心功能

### 1. RSS 订阅管理

**功能描述**：
- 通过 URL 添加订阅源
- 自动发现网站 RSS 链接
- 支持 RSS 2.0 / Atom / JSON Feed
- 文件夹分类管理
- OPML 导入/导出

**技术实现**：
- 后端：`rss-parser` 库解析 RSS/Atom
- 自动发现：`parseFeed()` 函数实现 HTML 解析
- 刷新策略：`refreshRoute` 定时抓取（默认 30 分钟）

**相关文件**：
- `api/src/lib/rss/parser.ts` — RSS 解析器
- `api/src/routes/feeds.ts` — 订阅源 CRUD
- `api/src/routes/refresh.ts` — 定时刷新
- `api/src/routes/opml.ts` — OPML 导入导出

### 2. 阅读全文（自动提取正文）

**功能描述**：
- 自动提取文章正文（类似 Safari 阅读模式）
- 移除广告、导航栏等无关内容
- 保留图片、链接、格式等有效内容
- 支持原文/提取内容切换

**技术实现**：
- 使用 `@extractus/article-extractor` 或 Readability.js 提取正文
- 后端实现 `extractArticle()` 服务
- 存储提取后的内容到 `articles.content` 字段
- 前端 `ArticleReader` 组件渲染

**数据模型**：
```typescript
// articles 表
{
  id: string,
  feedId: string,
  title: string,
  url: string,          // 原文链接
  content: string,      // 提取后的正文（HTML）
  summary: string,      // 摘要
  author: string,
  imageUrl: string,
  publishedAt: string,
  isRead: boolean,
  isStarred: boolean,
  isReadLater: boolean,
}
```

**API 端点**：
- `POST /api/articles/:id/extract` — 触发正文提取
- `GET /api/articles/:id` — 获取文章详情（包含提取后的正文）

**相关文件**：
- `api/src/lib/extractor/` — 正文提取服务（待实现）
- `api/src/routes/articles.ts` — 文章 API
- `frontend/src/components/app/article/ArticleReader.tsx` — 阅读器组件
- `mobile/lib/screens/article_detail_screen.dart` — 移动端阅读页

### 3. 文件夹分类

**功能描述**：
- 创建/编辑/删除文件夹
- 拖拽排序
- 展开/折叠文件夹
- 按文件夹筛选文章

**数据模型**：
```typescript
// folders 表
{
  id: string,
  userId: string,
  name: string,
  color: string,        // 文件夹颜色标识
  icon: string,         // 可选图标
  sortOrder: number,    // 排序权重
  isCollapsed: boolean, // 是否折叠
}
```

**相关文件**：
- `api/src/routes/folders.ts` — 文件夹 API
- `frontend/src/components/app/sidebar/FolderTree.tsx` — 文件夹树组件
- `mobile/lib/widgets/folder_tree.dart` — 移动端文件夹组件

### 4. 标记与滑动操作

**功能描述**：
- 标记已读/未读（支持批量）
- 收藏/取消收藏
- 稍后读
- **滑动操作**：左滑标记已读/未读，右滑收藏

**数据模型**：
```typescript
// articles 表状态字段
{
  isRead: boolean,      // 已读状态
  isStarred: boolean,   // 收藏状态
  isReadLater: boolean, // 稍后读状态
}
```

**API 端点**：
- `PUT /api/articles/:id` — 更新文章状态
- `POST /api/articles/batch` — 批量更新（标记全部已读等）

**前端交互**：
- **Web**：文章列表支持滑动（使用 `framer-motion` 或 `react-swipeable`）
  - 左滑：标记已读/未读
  - 右滑：收藏/取消收藏
  - 滑动阈值：100px
  - 滑动反馈：显示操作图标 + 背景色变化
  
- **iOS**：使用 `Dismissible` 或自定义滑动组件
  - 左滑：显示"已读/未读"按钮
  - 右滑：显示"收藏"按钮
  - 支持完整滑出快速操作

**相关文件**：
- `frontend/src/components/app/layout/ArticleList.tsx` — 文章列表（需添加滑动）
- `frontend/src/components/app/article/ArticleCard.tsx` — 文章卡片
- `mobile/lib/widgets/article_card.dart` — 移动端文章卡片

### 5. 双语翻译

**功能描述**：
- 调用谷歌翻译 API（免费版）
- 支持原文/译文/双语对照三种模式
- 缓存翻译结果
- 支持全文翻译

**技术实现**：
- 使用 Google Translate API (`translate.googleapis.com`)
- 无需 API Key，直接调用公共接口
- 后端 `translateRoute` 封装翻译请求
- 前端缓存翻译结果到 Zustand store

**API 端点**：
- `POST /api/translate` — 翻译文本

**相关文件**：
- `api/src/routes/translate.ts` — 翻译 API
- `frontend/src/components/app/article/ArticleReader.tsx` — 阅读器翻译 UI
- `frontend/src/lib/store/articleStore.ts` — 翻译缓存

## 常用命令

```bash
# 全局（在根目录执行）
pnpm dev              # 并行启动所有服务
pnpm dev:api          # 仅启动后端（端口 8787）
pnpm dev:frontend     # 仅启动前端（端口 4321）
pnpm build            # 构建所有子包
pnpm deploy           # 部署所有子包
pnpm lint             # 检查所有子包

# 后端（api/）
pnpm --filter api dev
pnpm --filter api deploy
pnpm --filter api db:migrate:local   # 本地数据库迁移

# 前端（frontend/）
pnpm --filter frontend dev
pnpm --filter frontend deploy

# 移动端（mobile/）
cd mobile && flutter run                      # 运行调试
cd mobile && flutter build ios               # 构建 iOS
cd mobile && flutter build ios --release     # 发布构建
cd mobile && flutter pub get                 # 获取依赖
```

## API 端点总览

### 认证
- `POST /api/auth/*` — Better Auth 认证（注册/登录/登出/会话）

### 文件夹
- `GET /api/folders` — 获取文件夹列表
- `POST /api/folders` — 创建文件夹
- `PUT /api/folders/:id` — 更新文件夹
- `DELETE /api/folders/:id` — 删除文件夹

### 订阅源
- `GET /api/feeds` — 获取订阅源列表
- `POST /api/feeds` — 添加订阅源
- `PUT /api/feeds/:id` — 更新订阅源
- `DELETE /api/feeds/:id` — 删除订阅源

### 文章
- `GET /api/articles` — 获取文章列表（支持筛选、分页）
- `GET /api/articles/:id` — 获取文章详情
- `PUT /api/articles/:id` — 更新文章状态
- `POST /api/articles/:id/extract` — 提取正文（待实现）
- `POST /api/articles/batch` — 批量操作（待实现）

### OPML
- `GET /api/opml/export` — 导出 OPML
- `POST /api/opml/import` — 导入 OPML

### 刷新
- `POST /api/refresh` — 手动触发 RSS 刷新

### 翻译
- `POST /api/translate` — 翻译文本

## 开发规范

### 后端 (api/)

**技术栈**：Hono + Drizzle ORM + Cloudflare D1

**代码规范**：
- 所有路由返回统一格式：`{ success: boolean, data?: T, error?: string }`
- 使用 Zod 验证请求参数
- 使用 `authMiddleware` 保护需要登录的接口
- 禁止在 api/ 中使用 `export const runtime = "edge"`

**数据库访问**：
```typescript
import { getDb } from "../db";
const db = getDb(c.env.DB);
```

**Cloudflare Bindings**：
- `c.env.DB` — D1 数据库
- `c.env.COLLECT_KV` — KV 存储（缓存）
- `c.env.COLLECT_R2` — R2 存储（文件）

修改 bindings 后运行：
```bash
pnpm --filter api cf-typegen
```

### 前端 (frontend/)

**技术栈**：Astro 6 + React Islands + Tailwind CSS v4 + shadcn/ui

**代码规范**：
- Astro 页面：`src/pages/`（SSR/SSG）
- React Islands：`src/components/app/`（客户端交互）
- API 客户端：`src/lib/api.ts`（调用 collect-api）
- 状态管理：Zustand stores（`src/lib/store/`）

**样式规范**：
- 使用 CSS 变量定义主题色
- 支持深色/浅色模式切换
- 主色调：`--accent`（品牌色）

**React Islands**：
```astro
<Component client:load />
```

### 移动端 (mobile/)

**技术栈**：Flutter 3 + Riverpod + go_router

**代码规范**：
- 状态管理：Riverpod (`flutter_riverpod`)
- 路由：`go_router` 声明式路由
- HTTP 客户端：Dio
- API 地址：`--dart-define=API_BASE_URL=<url>` 注入

**iOS 优先体验**：
- 使用 Cupertino 风格组件
- 支持 iOS 滑动返回手势
- 支持深色模式（跟随系统）
- 支持动态字体大小

**API 调用**：
```dart
final dio = Dio();
dio.options.baseUrl = const String.fromEnvironment('API_BASE_URL');
```

**Cookie 管理**：
Better Auth session 通过 Cookie 传递，移动端需手动管理 Cookie：
```dart
// 登录后保存 Cookie
final cookie = response.headers['set-cookie'];
await storage.write(key: 'session', value: cookie);

// 请求时附加 Cookie
dio.options.headers['Cookie'] = await storage.read(key: 'session');
```

## 待实现功能清单

### 高优先级
- [ ] 正文自动提取服务（`@extractus/article-extractor`）
- [ ] 文章批量操作 API
- [ ] Web 端文章列表滑动操作
- [ ] iOS 端文章列表滑动操作
- [ ] 翻译缓存优化（KV 存储）

### 中优先级
- [ ] 阅读进度保存
- [ ] 搜索功能（全文搜索）
- [ ] 智能文件夹（按关键词自动分类）
- [ ] 文章分享功能
- [ ] 离线阅读（PWA / Flutter 离线缓存）

### 低优先级
- [ ] 键盘快捷键（Vim 风格）
- [ ] 文章标注/笔记
- [ ] 阅读统计
- [ ] 邮件订阅摘要

## 参考资源

### RSS 相关
- [RSS 2.0 规范](https://www.rssboard.org/rss-specification)
- [Atom 规范](https://tools.ietf.org/html/rfc4287)
- [JSON Feed](https://www.jsonfeed.org/)

### 正文提取
- [@extractus/article-extractor](https://github.com/extractus/article-extractor)
- [Mozilla Readability](https://github.com/mozilla/readability)
- [Mercury Parser](https://github.com/postlight/mercury-parser)

### 翻译 API
- [Google Translate API (Free)](https://translate.googleapis.com)

### 设计参考
- [Reeder 5](https://www.reeder.app/)
- [Inoreader](https://www.inoreader.com/)
- [NetNewsWire](https://netnewswire.com/)

---
> Source: [redisread/folio](https://github.com/redisread/folio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
