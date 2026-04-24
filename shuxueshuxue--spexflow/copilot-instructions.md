## spexflow

> - 永远遵循最小实现原则；不做“未来可能用到”的扩展。

# AGENTS.md (spec-flow scope)

## 原则

- 永远遵循最小实现原则；不做“未来可能用到”的扩展。
- Debug 用 print/console/日志把现状打出来，再判断；不要用“match/猜测式”调试。
- Fail loudly：不要吞错、不要静默 fallback。
- 只在真正棘手处加注释，且使用格式：`@@@title - explanation`。

## 技术约定

- 前端：React + TypeScript + Vite；画布用 `@xyflow/react`（React Flow）。
- 后端：Express + TypeScript，入口 `server/index.ts`，用 `tsx watch` 跑。
- 模块：保持 ESM（`"type": "module"`）。

## 常用命令

- 安装：`pnpm install`
- 开发：`pnpm dev`（并发启动 web + server）
- 构建：`pnpm build`
- 后端健康检查：`curl http://localhost:3001/api/health`

## 常见问题

- 如果运行 `pnpm ...` 触发 corepack 报错 `Cannot find matching keyid`：直接在当前 Node 版本下安装 pnpm（绕过 corepack）：
  - `npm i -g pnpm@9.15.9 --force`（如果提示 `EEXIST` file already exists）
  - `hash -r`
  - `pnpm -v`
  - 仍不生效就先打印确认：`which -a pnpm`

---
> Source: [shuxueshuxue/SpexFlow](https://github.com/shuxueshuxue/SpexFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
