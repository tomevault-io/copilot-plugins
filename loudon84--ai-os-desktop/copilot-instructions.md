## 007-sync-project-docs

> 代码计划或功能实现完成后，同步更新 AGENTS.md 与 docs/ 文档（含 docs/renderer/）


# Post-Implementation Docs Sync

当**代码计划全部阶段完成**、**功能实现收尾**、或用户明确要求更新文档时：

1. **立即加载并执行** skill：`.agents/skills/sync-project-docs/SKILL.md`
2. 基于 `git diff` / 本次变更文件，按 skill 内 [doc-map.md](.agents/skills/sync-project-docs/doc-map.md) 与 [doc-tree.md](.agents/skills/sync-project-docs/doc-tree.md) **增量**更新相关段落

## 必查文档（按变更类型选子集，非每次全改）

### 核心层

- `AGENTS.md` — 版本、目录地图、Preload API、路由、**文档体系**、按功能跳转
- `docs/INDEX.md` — 版本特性、核心目录、**Renderer 文档**链接
- `docs/READING_GUIDE.md` — 分阶段阅读链、功能域阅读小节
- `docs/ARCHITECTURE.md` — 进程模型、布局演进、数据流（摘要不重复 IPC 全表）
- `docs/API_CONTRACTS.md` — IPC 单一事实源（新/改 channel 必改）

### Renderer 层（`src/renderer/**` 或 workspace 路由变更时）

- `docs/renderer/INDEX.md` — registry 启用状态、顶层结构
- `docs/renderer/screens/INDEX.md` + `docs/renderer/screens/<Screen>.md` — Screen 详情
- `docs/renderer/screens/web-operator/*.md` — WebOperator 子域
- `docs/renderer/workspace/*.md` — registry / secondary-nav / 分发
- 按需：`APP_STARTUP.md`、`MAIN_LAYOUT.md`、`WORKSPACE_ROUTING.md`、`COMPONENTS.md`、`HOOKS.md`、`PRELOAD_API_USAGE.md`

## 执行步骤

```
git --no-pager diff --stat && git --no-pager diff --name-only
→ 勾选 skill 内变更分类 checklist
→ 查 doc-map「变更类型 → 最低更新集」
→ 增量编辑（禁止凭记忆编造 IPC / 路径 / 版本号）
→ 自检：路径存在、IPC 与源码一致、AGENTS 与 INDEX 版本对齐
→ 输出「文档同步摘要」
```

文档更新是实现的**最后一步**，与 typecheck / test 同级；未完成文档同步不得宣告任务完成。

## 跳过条件

- 仅 typo / 注释 / 格式化，无行为或契约变化
- 用户明确说「不要改文档」

## 默认不同步（除非用户明确要求）

- `docs/MODULES.md`
- `docs/code-assets/`、`docs/memory-bank/`、`docs/specs/`、`docs/superpowers/`
- 大段复制粘贴源码到文档

## 禁止

- 凭记忆编造 IPC channel、文件路径或版本号
- 把 Renderer Screen 细节全部灌进 ARCHITECTURE（应写 `docs/renderer/`）
- 因文档同步而改业务代码

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
