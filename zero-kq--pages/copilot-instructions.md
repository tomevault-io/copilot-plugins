## pages

> 本文档为 Claude Code (claude.ai/code) 在本项目中工作提供指导。

# CLAUDE.md

本文档为 Claude Code (claude.ai/code) 在本项目中工作提供指导。

## 项目概述

这是一个使用 **PaperMod 主题** 的 **Hugo 静态站点**。站点通过 GitHub Actions 在推送到 main 分支时自动部署到 **GitHub Pages**。

## 语言偏好

与用户交流时使用**中文**。

## 用户偏好

1. 只修改 Hugo 模板和 CSS，不改动主题核心代码
2. 修改模板或样式前，先使用preview.html提供静态 HTML 预览效果供确认，确认后再更新实际项目文件。

## 命令

### 开发

```bash
# 本地启动站点并实时重载
hugo server

# 为生产环境构建站点
hugo

# 构建包含草稿的站点（用于预览未发布的内容）
hugo --buildDrafts
```

### GitHub Actions 部署
站点会在推送到 main 分支时自动部署。
默认使用 Hugo 版本 `0.146.0`。

## 文档任务规则
- 生成、更改和完善文档前必须先读取`demo.md`，按`demo.md`要求进行
- 不允许跳过此步骤直接参考已有文档的格式
- 在修改文档时，不能篡改已有的文字和代码内容，只对格式和布局进行优化

## 架构

### 目录结构

- `content/` - Markdown 内容文件（文章、页面）
- `layouts/` - 自定义 Hugo 模板（覆盖主题）
- `themes/PaperMod/` - 主题文件（本地克隆，非 git 子模块）
- `static/` - 静态资源（图片、CSS、JS）
- `assets/` - 构建资源（由 Hugo Pipes 处理）
- `data/` - Hugo 数据文件（YAML、JSON、TOML）
- `i18n/` - 国际化文件
- `archetypes/` - 内容脚手架模板

### 配置

- `hugo.toml` - Hugo 主配置
- `.github/workflows/gh-pages.yml` - GitHub Actions 工作流

### 主题自定义

要自定义 PaperMod，在 `layouts/` 中创建与主题结构相同的覆盖文件（如 `layouts/_default/terms.html` 覆盖分类页面）。

自定义 CSS 放入 `assets/css/extended/` 目录，主题会自动加载（无需在配置中引用）。

当前已覆盖的模板：
- `layouts/_default/terms.html` - 分类/标签页面（卡片样式）
- `assets/css/extended/terms.css` - 分类卡片样式

### Mermaid 图表支持

PaperMod 不内置 Mermaid.js，需手动配置：
- `layouts/partials/extend_head.html` - 引入 Mermaid CDN 和初始化配置
- `layouts/_default/_markup/render-codeblock.html` - 代码块渲染模板，将 mermaid 代码包裹为 `<div class="mermaid">`

使用方式：Markdown 中直接写 ` ```mermaid ` 代码块即可

### Hugo Taxonomy 数据获取

**Categories（分类）：**
```go
{{- range .Data.Terms.Alphabetical }}
{{- with site.GetPage (printf "/%s/%s" $.Type .Name) }}
<a href="{{ .Permalink }}">{{ .Name }}</a>
{{- end }}
{{- end }}
```

**Tags（标签）：**
```go
{{- range site.Taxonomies.tags.ByCount }}
<a href="{{ .Page.RelPermalink }}">{{ .Page.Title }} <span>{{ .Count }}</span></a>
{{- end }}
```

### PaperMod CSS 变量

常用变量：`--gap`, `--radius`, `--entry`, `--primary`, `--secondary`, `--tertiary`, `--border`, `--header-height`, `--theme`

所有颜色使用 CSS 变量，自动适配亮色/暗色主题。

### 三栏侧边栏布局

文章详情页使用三栏布局：
- `layouts/_default/single.html` - 三栏布局模板
- `layouts/partials/sidebar-left.html` - 左侧固定侧边栏（头像+导航+社交图标）
- `layouts/partials/sidebar-right.html` - 右侧浮动侧边栏（TOC+最近更新+热门标签）
- `assets/css/extended/custom.css` - 三栏布局样式

头像图片放在 `static/images/avatar.jpg`，路径配置为 `images/avatar.jpg`

### Hugo 配置注意点

- `profileMode.enabled = true` 会触发 PaperMod 内置的首页博主信息展示，侧边栏仅需配置 title/description/imageUrl
- 静态资源路径用 `relLangURL` 或 `printf "%s%s" site.BaseURL .` 拼接，避免 baseURL 末尾斜杠导致的重复斜杠问题
- `partial "social_icons.html"` 需要传入 align 参数：`partial "social_icons.html" (dict "align" "center")`
- CSS 覆盖主题宽度限制时用 `body:not(.list) .main` 选择器，只影响文章详情页

### 搜索功能

PaperMod 内置 Fuse.js 模糊搜索，支持标题、摘要、正文内容搜索。

**启用搜索：**
1. 在 `hugo.toml` 中配置 `[outputs] home = ["HTML", "JSON"]`
2. 创建 `content/search.md`，内容：`layout: "search"`
3. 在导航菜单中添加搜索页面链接

**搜索页面模板：** `themes/PaperMod/layouts/_default/search.html`

**搜索快捷键：** `Alt + /`

### Hugo 清理构建

修改 CSS 或模板后需要完全清理缓存：
```bash
rm -rf public resources && hugo
```

### CSS 资源加载

`assets/css/extended/` 下的 CSS 需要在 `layouts/partials/extend_head.html` 中手动引入：
```go
{{- $customCSS := resources.Get "css/extended/custom.css" }}
{{- if $customCSS }}
<link rel="stylesheet" href="{{ $customCSS.RelPermalink }}">
{{- end }}
```

### Layout 覆盖优先级

Hugo 按以下顺序查找模板：`layouts/<section>/list.html` > `layouts/_default/list.html` > 主题
页面 section（如 `/posts/`）有专属模板时需单独处理

### Markdown 禁用原始 HTML

Hugo 默认禁用原始 HTML，复杂内容用 partial 渲染，不要写在 Markdown 中

### 关于页布局

关于页不使用三栏布局，创建 `layouts/about/single.html` 单独处理

---
> Source: [Zero-Kq/Pages](https://github.com/Zero-Kq/Pages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
