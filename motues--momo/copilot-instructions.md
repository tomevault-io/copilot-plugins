## momo

> 一个极简的本地 CMS 管理后台，用于直接管理 Momo 博客的文章内容（`src/content/blog/**/*.md`）。

# Momo CMS

一个极简的本地 CMS 管理后台，用于直接管理 Momo 博客的文章内容（`src/content/blog/**/*.md`）。

## 快速开始

```bash
# 1. 安装依赖（cms 是 pnpm workspace 成员，根目录安装即可）
pnpm install

# 2. 启动 CMS（根目录执行）
pnpm cms
```

启动后访问 **http://localhost:5188**（单端口，前端与 API 由 Hono + Vite 一体提供，支持 HMR）。

> 保存文章即直接写入博客内容目录，博客 dev server（`pnpm dev`）会实时热更新。

## 功能

- **概览页**（`#/`）：文章总数 / 已发布 / 草稿 / 置顶 / 分类数 / 正文总字数统计，分类分布条形图，语言版本覆盖（中英双语），最近文章列表
- **文章列表**（`#/list`）：搜索、分类筛选、草稿/已发布筛选、语言徽章；支持**卡片 / 表格**两种视图模式（localStorage 记忆选择）与**多种排序**（默认置顶+日期 / 发布日期升降序 / 标题 / 路径 / 分类，中文按拼音排序）；表格右侧**行内操作**（置顶/取消置顶、草稿/发布切换、删除）
- **Markdown 编辑器**：frontmatter 表单 + 正文源码，左侧编辑右侧**实时预览**（防抖 500ms）；编辑区上方**快速插入工具栏**（加粗/斜体/行内代码/链接/图片/引用、代码块/Typst、行内/块公式、提示块（note/tip/important/caution/warning）、GitHub/音乐卡片、注音/折叠/彩虹/下划线），支持选中文本包裹与光标定位
- **完整自定义语法预览**：与博客渲染管线一致（见下方语法表）
- **多语言版本**：同路径 `zh-cn.md` / `en.md` 标签页切换，可新建缺失的语言版本
- **slugId**：文章/评论标识元数据（如 `momo/xxx`），与文件夹位置解耦，保存时**不会**改变文件夹；仅当 slugId 与当前文件夹路径一致（CMS 新建的文章）且被修改时才整体移动文件夹
- **封面图上传**：图片直接保存到文章文件夹，自动填写 `image: ./xxx.png`
- **删除保护**：删除前二次确认，未保存内容离开页面时提醒
- **深色模式**：跟随系统

## 技术架构

```
cms/
├── package.json          # 独立 workspace 包（依赖：hono、@hono/vite-dev-server、unified 插件链…）
├── vite.config.ts        # devServer 插件：Hono 应用作为 SSR 入口，单端口一体运行
├── index.html            # SPA 外壳
├── server/               # Hono 服务端（Node ESM，由 Vite SSR 加载）
│   ├── index.mjs         # 应用入口：/api/* 路由 + SPA 回退 + 静态资源
│   ├── store.mjs         # 博客文章文件统一读写层（CRUD / slugId 元数据 / 分类统计）
│   ├── articles.mjs      # GET/POST/PUT/DELETE /api/articles/*
│   ├── preview.mjs       # POST /api/preview：复用博客全部 remark/rehype 插件
│   ├── upload.mjs        # POST /api/upload：封面图上传（multipart）
│   ├── meta.mjs          # GET /api/meta：分类统计
│   ├── stats.mjs         # GET /api/stats：概览页统计（字数 / 语言覆盖 / 最近文章）
│   ├── blog-content.mjs  # GET /blog-content/*：文章文件夹静态资源（预览图片）
│   └── prose.css         # 预览正文样式（与博客 markdown.css 一致的精简版）
└── src/                  # 前端（纯 TypeScript SPA，hash 路由，无框架）
    ├── main.ts / router.ts / api.ts / types.ts / styles.css
    └── pages/ OverviewPage.ts（概览）、ListPage.ts（列表：卡片/表格）、EditorPage.ts、
              header.ts（顶栏导航）、new-article.ts（新建文章弹窗）
```

- **API 端口**：5188（唯一端口，Vite dev server 内嵌 Hono）
- **预览管线**：直接复用 `../src/plugins/*.mjs`（remark-typst、remark-directive-rehype、remark-combined、admonition、github/music/quote 卡片、figure 插件）+ KaTeX + shiki 高亮（one-dark-pro 主题），与博客 astro.config.mjs 的插件顺序一致，跳过仅构建期需要的 reading-time 与 LQIP 插件。

## API 一览

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/api/articles?q=&category=&draft=` | 文章列表（分组、筛选、搜索） |
| GET | `/api/articles/:path` | 读取文章全部语言版本 |
| POST | `/api/articles` | 新建文章 `{ path, lang }` |
| PUT | `/api/articles/:path/:lang` | 保存 `{ data, body }`；仅当 slugId 与路径一致且被修改时移动文件夹 |
| DELETE | `/api/articles/:path` | 删除整篇文章（文件夹） |
| POST | `/api/preview` | 实时预览 `{ data, body, base }` → 完整 HTML 文档 |
| POST | `/api/upload` | 上传封面图（multipart: file + path） |
| GET | `/api/meta` | 分类统计 |
| GET | `/api/stats` | 概览统计（总数/发布/草稿/置顶/分类/字数/语言/最近文章） |

## 自定义语法速查（预览与博客一致）

| 语法 | 说明 |
|---|---|
| `:::note{name="提示"}`…`:::`（note/tip/important/caution/warning） | Alert 提示块（块级容器，`{name="..."}` 可选） |
| `::github{repo="owner/repo"}` | GitHub 仓库卡片 |
| `::music{id="歌曲ID"}` | 网易云音乐卡片 |
| `::quote[内容]` | 居中引用组件 |
| `$...$` / `$$...$$` | KaTeX 数学公式 |
| ```` ```typst ```` | Typst 代码块 → SVG 渲染 |
| `{中文}(zhong wen)` | 注音（Ruby） |
| `!!内容!!` | 折叠内容（hover 显示） |
| `==内容==` | 彩虹文字 |
| `++内容++` | 下划线 |
| `![标题图片](./cover.jpg)` | 图片 + figure 标题（`title` 属性作为 figcaption） |

## 注意事项

- 内容为本地可信数据；预览 iframe 使用 `sandbox="allow-scripts"` 沙箱。
- Typst 首次编译需加载原生编译器，约 1-3 秒；失败时预览区显示错误信息。
- 修改 `server/` 目录下的代码后需要重启 `pnpm cms`（服务端模块不参与 HMR）。
- 博客根目录相对路径图片（`/images/xxx.png`）在预览中不会加载（仅本地文件相对路径可用）。

## 测试

`smoke.mjs` 是基于 jsdom 的端到端冒烟测试（真实点击「新建文章」→ 创建 → 编辑 → 实时预览 → 保存 → 删除）。需先启动服务再运行：

```bash
pnpm cms        # 终端 1：启动服务
pnpm smoke      # 终端 2：运行冒烟测试
```

---
> Source: [Motues/Momo](https://github.com/Motues/Momo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
