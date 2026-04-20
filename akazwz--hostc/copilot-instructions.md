## hostc

> - **Monorepo** 管理，使用 pnpm。所有应用在 `apps/`，共享包在 `packages/`（目前为空）。

# AGENT INSTRUCTIONS: hostc Monorepo

## 项目结构

- **Monorepo** 管理，使用 pnpm。所有应用在 `apps/`，共享包在 `packages/`（目前为空）。
- **主要子项目：**
  - `apps/workers`：Cloudflare Worker，TypeScript + Wrangler + Durable Objects。
  - `apps/cli`：Node.js CLI 工具。

## 常用命令（根目录执行）

| 命令                  | 说明                       |
|----------------------|----------------------------|
| `pnpm install`       | 安装所有依赖               |
| `pnpm dev:workers`   | 启动 Workers 本地开发       |
| `pnpm deploy:workers`| 部署 Workers 到 Cloudflare |
| `pnpm build:cli`     | 构建 CLI 工具               |

> 子项目有更多命令，详见各自的 `package.json`。

> 也可以直接使用 pnpm filter 语法，例如 `pnpm -F workers dev`、`pnpm -F hostc build`。

## 约定与建议

- 新增共享代码请放在 `packages/`，并通过 pnpm workspace 依赖引用。
- 详细 Worker 相关约定见 `apps/workers/AGENTS.md`。
- 详细 CLI 相关约定见 `apps/cli/` 目录。

## 参考文档

- [pnpm workspace](https://pnpm.io/workspaces)
- [Cloudflare Workers](https://developers.cloudflare.com/workers/)

---
---
各子项目（如 apps/workers、apps/cli）有专属 AGENTS.md，具体开发约定请优先参考对应目录下的说明文件。
如需补充全局约定或有特殊需求，请在本文件补充说明。

---
> Source: [akazwz/hostc](https://github.com/akazwz/hostc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
