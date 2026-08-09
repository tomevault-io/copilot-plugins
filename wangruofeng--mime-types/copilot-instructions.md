## mime-types

> 纯前端、单文件的 MIME 类型速查工具。面向人类的说明见 `README.md`；本文件记录**改代码时必须知道的红线与约定**。项目与同级目录 `../json-viewer`、`../xml-viewer` 保持同一套设计与代码风格。

# mime-types — 项目约定（给 AI 协作时参考）

纯前端、单文件的 MIME 类型速查工具。面向人类的说明见 `README.md`；本文件记录**改代码时必须知道的红线与约定**。项目与同级目录 `../json-viewer`、`../xml-viewer` 保持同一套设计与代码风格。

## 红线（不要破坏）

- **单文件、零构建、零依赖**：全部 HTML / CSS / JS 都在 `index.html` 一个文件里。**禁止**引入打包器（Vite/Webpack 等）、框架（React/Vue 等）、npm 依赖或构建步骤。新增能力用原生 JS 实现。
- **纯前端、无后端**：所有查询与复制操作在浏览器本地完成，不上传任何数据。
- 主题用根元素 `html.dark` 类切换，配色全部走 `:root` / `html.dark` 里的 CSS 变量。新增颜色请复用已有变量，不要硬编码。
- 数据以内嵌 JS 常量形式存在，扩展名必须唯一。

## 文件结构

- `index.html` — 全部代码（结构 + 样式 + 脚本 + 内嵌 MIME 数据）。
- `favicon.svg` — 蓝紫渐变圆角方块 + 白色 `M:` 标志。
- `.github/workflows/deploy.yml` — GitHub Pages 部署：仓库根即站点根，push 到 `main` 自动部署。
- `README.md` — 面向用户的功能说明与在线访问地址。
- `CLAUDE.md` — 本文件，项目红线与约定。

## 数据约定

- `MIME_DATA` 数组：每条 `{ ext, mime, cat, desc }`。
- `cat` 取值固定为：`docs`（文档）、`image`（图片）、`audio`（音频）、`video`（视频）、`font`（字体）、`archive`（压缩）、`code`（代码）、`data`（数据）。
- 扩展名（`ext`）必须全局唯一，禁止重复。
- 多段扩展名（如 `tar.gz`）需要在 `MULTI_PART_EXTS` 中登记，供文件名扩展名提取优先匹配。

## UI 约定

- 页头左侧 `M:` logo mark，右侧 GitHub 源码链接与主题切换按钮。
- 工具栏：搜索框 + 分类分段控件（全部 / 文档 / 图片 / 音频 / 视频 / 字体 / 压缩 / 代码 / 数据）+ 统计。
- 结果表格：扩展名 / MIME 类型 / 类别 / 说明 / 复制按钮。
- 按钮文案保持极简，图标优先。

## 在线地址

已部署到 GitHub Pages：https://blog.wangruofeng007.com/mime-types/

---
> Source: [wangruofeng/mime-types](https://github.com/wangruofeng/mime-types) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
