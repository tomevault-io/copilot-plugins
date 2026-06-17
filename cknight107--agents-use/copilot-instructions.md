## agents-use

> - 具体规则、背景知识、长期约束、隐性约定统一放进 `docs/`

# AGENTS.md

## 核心原则
- 具体规则、背景知识、长期约束、隐性约定统一放进 `docs/`

## 仓库工作流
- 本仓库采用“先查文档，再做变更”的工作流。
- 涉及功能开发、重构、接口调整、目录调整时，先确认是否已有对应 `change` 记录；没有则先补 `docs/changes/` 再开始实现。
- 若实现需要突破既有规则，先更新 `docs/rules/` 中对应文档，再修改代码，不允许代码长期偏离文档。

## AI 进入仓库后的必读顺序
1. 先读 `docs/rules/README.md`，了解规则索引。
2. 再读 `docs/rules/project-context.md`，确认工作区结构、分层职责和依赖方向。
3. 按任务需要继续读：
   - 代码风格相关：`docs/rules/code-style.md`
   - TypeScript 约束：`docs/rules/ts.md`
   - 安全边界：`docs/rules/security.md`
4. 涉及业务背景、术语、历史上下文时，再读 `docs/knowledge/README.md` 及其子文档。
5. 涉及需求或实现变更时，查看 `docs/changes/README.md`、对应 `docs/changes/templates/` 和 `docs/changes/log/`。

## 没有 change 是否允许开始开发
- 不允许直接开始正式开发。
- 例外仅限极小改动：修正BUG、纯文案修正、注释修正、格式整理、不会影响行为的微小修补。
- 只要涉及行为变化、接口变化、数据结构变化、依赖变化、目录结构变化，就必须先有对应 `change`。

## 受保护目录
- `docs/rules/`：长期规则来源。没有明确理由不要改；若要改，必须让规则变更先于代码变更。
- `docs/knowledge/`：项目知识来源。只补充事实和背景，不写执行指令。
- `docs/changes/`：变更记录来源。新增开发前优先在这里补齐上下文。
- `packages/database/src/generated/`：生成产物目录。不要手改，必须通过生成命令更新。
- `node_modules/`：外部依赖目录。不要手改。

## 主要命令入口
- 根级统一入口：
  - `pnpm dev`
  - `pnpm build`
  - `pnpm typecheck`
  - `pnpm lint`
  - `pnpm test`
- 这些命令最终通过 `scripts/workspace-task-runner.mjs` 分发到各工作区。
- 仅在需要单独定位问题时，才进入具体工作区执行局部命令。

## 工作区定位提示
- `apps/web`：前端页面、路由、展示和交互。
- `apps/api`：后端接口、BFF、鉴权、协议适配。
- `packages/shared`：共享类型、Schema、契约。
- `packages/ui`：共享 UI 组件与样式能力。
- `packages/ai-sdk`：AI 接入与运行时桥接。
- `packages/database`：Prisma 与数据访问边界。

## 执行要求
- 改代码前，先判断应该看哪份 `docs/` 文档，不要跳过。
- 发现文档缺失时，优先补 `docs/`，再继续实现。
- 除非任务明确要求，否则不要把规则全文、背景知识或长篇说明塞回 `AGENTS.md`。

---
> Source: [cKnight107/agents-use](https://github.com/cKnight107/agents-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
