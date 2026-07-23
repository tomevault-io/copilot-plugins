## rico-md

> > 本文件由 Claude Code 自动读取。目的是让 AI 快速理解代码组织方式，避免改错东西。

# Rico MD — Claude Code 协作文档

> 本文件由 Claude Code 自动读取。目的是让 AI 快速理解代码组织方式，避免改错东西。
> 产品定义见 docs/PRD.md，设计规范见 docs/DESIGN.md。

## 这个项目是什么

面向微信公众号的纯前端 Markdown 编辑器。无构建步骤，静态文件直接运行。

## 怎么跑

```bash
python -m http.server 8080   # 或 ./start.sh
# http://localhost:8080
```

## 代码怎么组织的

入口是 `index.html`，加载 `assets/scripts/main.js` 启动 Vue 应用。

```
main.js          →  应用入口。Vue 实例、状态管理、文档生命周期、工具栏交互
core/            →  不依赖 UI 的基础能力（渲染、图片、粘贴、高亮）
export/          →  复制/导出策略（公众号、X/Twitter、公式各一个文件）
storage/         →  localStorage 读写（偏好设置）
ui/              →  界面逻辑（主题管理、面板、Toast、代码主题）
styles/          →  CSS（base、editor、panel、about、themes/）
```

改功能时先定位到对应目录，不要在 main.js 里堆逻辑。

## 改动时要注意的

### 存储兼容性（最容易出问题）

这些 localStorage 键已有用户数据，不能改名、不能改数据结构：

`currentStyle` `markdownInput` `documents` `activeDocumentId` `codeBlockSettings`

IndexedDB 数据库名 `WechatEditorImages` 也不能改。

`img://` 协议用于编辑器内图片引用，渲染时替换为 blob URL，复制时替换为 Base64。这三个环节的替换逻辑分散在不同文件中，改图片处理时要确保三个环节都对。

### 文档删除的边界

删除操作必须保证 `documents.length >= 1`。删完最后一篇后要自动新建空白文档，`activeDocumentId` 必须指向有效文档。弹窗初始状态 `deleteConfirm.show === false`。

### 主题系统

正文主题和代码主题是独立的，分别管理：
- 正文主题：`styles/themes/` 目录下的 JS 文件，由 `ui/theme-manager.js` 加载
- 代码主题：`ui/code-themes.js`，面板中有迷你预览卡片

新增主题时两个系统分别处理，不要混在一起。

### 公众号复制的兼容性

公众号不支持 CSS Grid、Flexbox（部分）、CSS Variables。复制到公众号时 `clipboard-exporter.js` 会做转换：
- Grid → Table
- 样式全部内联化
- 图片转 Base64
- 公式从 KaTeX 转 MathJax SVG

改复制相关逻辑后，必须实际粘贴到公众号编辑器验证，预览正常不代表复制后正常。

### 样式文件

CSS 拆分为 base / editor / panel / about 四个文件，不要把样式写回 index.html 的 `<style>` 标签里。

## 不要做的事

- 不要引入 npm / Vite / Webpack 等构建工具
- 不要引入后端
- 不要把分散在 core/、export/ 等目录的代码合并回单文件
- 不要改动已有的 localStorage 键名和数据结构

---
> Source: [ricocc/rico-md](https://github.com/ricocc/rico-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
