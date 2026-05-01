## pi-book

> 本目录是《pi 的设计艺术：构建生产级 Coding Agent 的架构决策》一书的写作空间。

# pi-book 写作指南

## 项目概述

本目录是《pi 的设计艺术：构建生产级 Coding Agent 的架构决策》一书的写作空间。

全书以 pi-mono 项目的**设计决策**为骨架，源码为佐证，面向有工程经验、想真正理解 agent 系统设计的开发者。

## 大纲

完整大纲见 [outline.md](outline.md)。Codex v1 大纲见 [outline-v1.md](outline-v1.md) 供参考。

## 章节 Spec（任务合约）

每篇/每组章节有一个 agent-spec 任务合约，定义了写作要求、验收标准和取舍约束。
**写任何章节前，必须先读对应的 spec。**

| Spec 文件 | 覆盖章节 | 预估工时 | 依赖 |
|-----------|---------|---------|------|
| [project.spec.md](specs/project.spec.md) | 全书约束 | - | - |
| [ch01-prologue.spec.md](specs/ch01-prologue.spec.md) | 第 1 章：序章 | 0.5d | - |
| [ch02-03-layering.spec.md](specs/ch02-03-layering.spec.md) | 第 2-3 章：分层的纪律 | 1d | - |
| [ch04-07-pi-ai.spec.md](specs/ch04-07-pi-ai.spec.md) | 第 4-7 章：pi-ai 设计 | 2d | ch02-03 |
| [ch08-10-runtime.spec.md](specs/ch08-10-runtime.spec.md) | 第 8-10 章：Agent Runtime | 2d | ch04-07 |
| [ch11-14-product.spec.md](specs/ch11-14-product.spec.md) | 第 11-14 章：产品化 | 2d | ch08-10 |
| [ch15-18-extensibility.spec.md](specs/ch15-18-extensibility.spec.md) | 第 15-18 章：能力外置 | 1.5d | ch11-14 |
| [ch19-23-tools.spec.md](specs/ch19-23-tools.spec.md) | 第 19-23 章：工具设计 | 2d | ch08-10 |
| [ch24-27-ui.spec.md](specs/ch24-27-ui.spec.md) | 第 24-27 章：UI 层 | 1.5d | ch11-14 |
| [ch28-29-products.spec.md](specs/ch28-29-products.spec.md) | 第 28-29 章：产品化实证 | 1d | ch11-14 |
| [ch30-32-philosophy.spec.md](specs/ch30-32-philosophy.spec.md) | 第 30-32 章：设计哲学 | 1.5d | ch15-18, ch19-23 |
| [appendix.spec.md](specs/appendix.spec.md) | 附录 A-D | 0.5d | ch08-10, ch19-23 |

### 依赖关系（关键路径用粗线标注）

```
ch01-prologue (0.5d)
ch02-03-layering (1d) ═══> ch04-07-pi-ai (2d) ═══> ch08-10-runtime (2d) ═══> ch11-14-product (2d) ═══> ch15-18-extensibility (1.5d) ═══> ch30-32-philosophy (1.5d)
                                                          │                         │
                                                          ├──> ch19-23-tools (2d) ──┤──> appendix (0.5d)
                                                          │                         │
                                                          │                         ├──> ch30-32-philosophy
                                                          │
                                                          ├──> ch24-27-ui (1.5d)
                                                          └──> ch28-29-products (1d)
```

**关键路径**: ch02-03 → ch04-07 → ch08-10 → ch11-14 → ch15-18 → ch30-32（总计 10d）

## 写作规则

1. **每章以设计问题开头** — 不是"这个模块有什么"，而是"为什么要这样设计"
2. **每个关键决策标注取舍** — 得到什么、放弃什么
3. **源码只在需要时出场** — 引用格式：`packages/ai/src/api-registry.ts:66-78`
4. **中文写作，术语保留英文** — 如 provider, stream, tool call, compaction
5. **每章 3000-6000 字** — 不超过 8000 字
6. **代码块不超过 40 行** — 只展示设计核心，不贴完整实现
7. **图示使用 Mermaid** — 放在章节内，不创建单独图片文件
8. **章节文件命名** — `src/ch{NN}-{slug}.md`（mdbook 结构）
9. **目录结构由 `src/SUMMARY.md` 控制** — 修改章节顺序或添加新章节时同步更新

## 构建与预览

```bash
# 构建 HTML 书籍
cd pi-book && mdbook build

# 本地预览（自动刷新）
cd pi-book && mdbook serve --open
```

## 写作前的检查

```bash
# 读取对应 spec
agent-spec contract specs/ch04-07-pi-ai.spec.md

# 写完后验证 spec
agent-spec verify specs/ch04-07-pi-ai.spec.md
```

## 源码参考

写作时需要参考的 pi-mono 源码在上级目录：

- `../packages/ai/` — pi-ai 层
- `../packages/agent/` — pi-agent-core 层
- `../packages/coding-agent/` — pi-coding-agent 层
- `../packages/tui/` — pi-tui 层
- `../packages/mom/` — mom (Slack bot)
- `../packages/pods/` — pods (GPU 编排)
- `../packages/web-ui/` — Web UI

---
> Source: [ZhangHanDong/pi-book](https://github.com/ZhangHanDong/pi-book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
