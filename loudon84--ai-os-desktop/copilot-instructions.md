## 000-ai-coding-entry

> AI Coding 入口 — 任务前先读 docs/ai-coding 与 context-map 路由；Superpowers 工作流见 superpowers-workflow.md


# AI Coding 入口（000）

## 任务开始（强制）

1. 扫读 `AGENTS.md` 目录地图与 Preload 表（不读全文）
2. 打开 `docs/ai-coding/README.md` § Context Map 路由
3. 加载 **一个** `docs/ai-coding/context-map/<domain>.md`
4. 若涉 Superpowers 流程，读 `docs/ai-coding/superpowers-workflow.md`

归属不明 → `docs/ai-coding/context-map/00-routing.md`

## Token 节流

- **禁止**无差别读取 `prd/`、`docs/INDEX.md` 全文、`node_modules/`、`out/`
- L3 源码：计划或 context-map 点名后再 Read（单次 ≤5 文件）
- 结构查询优先 CodeGraph（见 `codegraph.mdc`）

## 与现有 Rules 关系

本 rule **不覆盖** 下列专项 rule，与之叠加：

| Rule | 场景 |
|------|------|
| `001-electron-architecture.mdc` | main / preload |
| `003-ipc-contract.mdc` | IPC |
| `005-hermes-desktop-domain.mdc` | 域词汇与边界 |
| `007-sync-project-docs.mdc` | 功能收尾文档 |
| `008-loudon-command-comments.mdc` | loudon 注释锁 |
| `workbuddy-product-line.mdc` | `screens/Hermes/**` |
| `no-wait-skipped.mdc` | 多阶段任务状态机 |

## Superpowers

- 创意/新功能 → `brainstorming` → `writing-plans` → 执行 skill
- 计划路径：`docs/superpowers/plans/`、`docs/superpowers/specs/`
- 模板：`docs/ai-coding/templates/`

## 验收

```bash
npm run typecheck
```

IPC / 逻辑变更加 `npm test`；Renderer eslint 区加 `npm run lint`。

## 忽略索引

构建产物与运行时路径见根目录 `.cursorignore`。

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
