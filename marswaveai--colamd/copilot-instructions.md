## colamd

> **Agent Native 的 Markdown 编辑器。**

# ColaMD

## 产品定位

**Agent Native 的 Markdown 编辑器。**

面向 Agent-to-Agent、Agent-to-Human 之间的新型协作方式。当前 v1.0 聚焦于核心体验：AI Agent 修改 .md 文件时，ColaMD 自动检测变化并实时显示最新内容，让人类和 Agent 的协作像结对编程一样流畅。

后续会随着 Agent 生态的发展持续迭代。

## 设计哲学

### 如非必要，勿增实体

这是 ColaMD 的第一原则。每增加一个 UI 元素、一个功能、一行代码，都要问：这是绝对必要的吗？默认答案是否。

- 不要工具栏（用户会用快捷键和 Markdown 语法）
- 不要侧边栏
- 不要状态栏
- 界面只有：标题栏（拖拽用）+ 编辑器
- 追求极致的简单，一个功能做到极致

### 核心功能优先级

1. **文件热更新**（核心卖点）— 外部 Agent 修改 .md 时自动刷新，实时看到 Agent 的工作
2. **所见即所得** — 输入 Markdown 即刻渲染为富文本
3. **主题系统** — CSS 主题，可导入自定义主题
4. **导出** — PDF、HTML

### 不做的事情

- 不做文件管理、文件树、工作区
- 不做知识库管理
- 不做云同步、协作编辑
- 不做笔记组织和标签系统
- 不加不必要的 UI 元素（工具栏、侧边栏等）

## 技术栈

- Electron（桌面跨平台）
- Milkdown（基于 ProseMirror 的 WYSIWYG Markdown 框架）
- TypeScript 严格模式
- electron-vite（构建）
- electron-builder（打包）

## 项目结构

```
src/
├── main/           # Electron 主进程
│   └── index.ts    # 窗口管理、文件 I/O、菜单、文件监听
├── preload/        # 安全 IPC 桥接
│   └── index.ts
└── renderer/       # 渲染进程
    ├── index.html
    ├── main.ts     # 入口，连接编辑器和 IPC
    ├── editor/     # Milkdown 编辑器核心
    ├── themes/     # CSS 主题 + 主题管理器
    └── env.d.ts
```

## 开发规范

- TypeScript 严格模式
- 编辑器核心与 UI 解耦
- 主题 CSS 与编辑器逻辑完全分离
- 代码简洁，不过度设计
- 每个新功能先问：这是必要的吗？

---
> Source: [marswaveai/ColaMD](https://github.com/marswaveai/ColaMD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
