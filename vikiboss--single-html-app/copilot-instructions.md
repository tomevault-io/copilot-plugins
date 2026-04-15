## single-html-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目说明

单文件 HTML 应用集合。每个应用是一个独立的 `.html` 文件，无构建步骤，直接在浏览器中打开运行。

- `index.html` — 首页，展示所有应用的导航卡片
- `readme.md` — 项目说明和应用列表
- `xxx.html` — 各个应用的实现文件

## 新应用开发流程

1. **创建新应用文件** — 在根目录下创建一个新的 `.html` 文件，例如 `my-app.html`。
2. **实现应用功能** — 在新文件中编写 HTML、CSS 和 JavaScript 代码，实现所需功能。
3. **更新导航和说明** — 参考下文“添加新应用”部分的说明。

## 新应用风格约束

### 游戏类

- 如果需要在移动端进行滑动，注意阻止游戏区域的默认滚动行为（如下滑刷新、侧滑返回）。
- 如果很复杂，可以考虑 canvas 或 WebGL 实现，确保性能和实现效果。
- 考虑合适的激励，比如成功后的 canvas-confetti 效果，增加用户成就感。

### 工具类

- 暂无

## 添加新应用

添加新应用时，**必须同时修改以下两处**：

1. **`index.html`** — 在 `APPS` 数组中添加新应用对象：

```js
const APPS = [
  {
    title: '应用名称',
    description: '简短描述',
    href: 'your-app.html',
    tags: ['标签'], // 可选，便于分类和搜索
    date: '2026-03-04', // 新建应用时的日期
  },
]
```

2. **`readme.md`** — 在应用列表表格中添加对应行。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vikiboss)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vikiboss)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
