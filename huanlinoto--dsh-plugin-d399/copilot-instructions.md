## dsh-plugin-d399

> 贪玩蓝鲸 — 当模型生成时右下角弹出小游戏菜单。纯客户端 UI 插件，host half 为空。

# dsh-d399 — Agent Guide

## Plugin overview

贪玩蓝鲸 — 当模型生成时右下角弹出小游戏菜单。纯客户端 UI 插件，host half 为空。

## Key conventions

- **Bundle form**: `cordis.patch.yml` inserts one plugin row; `package.json` has `dsh.bundle.patch`. No source patches to DSH staging.
- **Client-only**: the host half (`src/index.ts`) has an empty `apply`. All work is in the browser half (`src/client/`).
- **预构建 lib/**: `lib/` 入库，无 `prepare` 脚本（含 `@deepseek-ai/*` private peer → 必须预构建策略）。
- **Peer deps**: cordis + react + react-dom + `@deepseek-ai/dsh-client-runtime` + `@deepseek-ai/dsh-invariants`（host 提供）。零运行时 npm 依赖。
- **Game registry service**: `ctx.provide('d399Games', registry)` 暴露给第三方插件注册游戏。

## File responsibilities

| File | Role |
|------|------|
| `src/index.ts` | Host entry: `name`, 空 `apply` |
| `src/invariant.ts` | `./invariant` companion（空 installer） |
| `src/client/index.tsx` | Client entry: `inject = ['sessions']`，provide d399Games 服务 + 注册内置游戏 + 挂载 overlay |
| `src/client/registry.ts` | `D399GamesService` 实现 + `Game`/`GameProps` 类型 |
| `src/client/useSessionRunning.ts` | `useSyncExternalStore` 订阅 current session 的 `running` |
| `src/client/D399Overlay.tsx` | 主组件: teaser / menu / game modal（portal 到 document.body） |
| `src/client/games/types.ts` | `GameProps` 共享类型 |
| `src/client/games/index.ts` | `registerBuiltinGames`: wordle + match3 + mini catalog |
| `src/client/games/wordle/logic.ts` | Wordle 纯逻辑（evaluateGuess / computeStatus / validateGuess） |
| `src/client/games/wordle/words.ts` | 5 字母词表（答案词 + 接受猜测词） |
| `src/client/games/wordle/Wordle.tsx` | Wordle 组件 |
| `src/client/games/match3/logic.ts` | 消消乐纯逻辑（findMatches / trySwap / resolveCascades） |
| `src/client/games/match3/Match3.tsx` | 消消乐组件 |
| `src/client/games/webframe/{WebFrame,BookmarkPanel,bookmarks}.ts(x)` | 自定义网页书签（localStorage + 动态注册为 game） |
| `src/client/games/mini/shared.ts` | 纯函数助手 + 词表/题库/谜语（无 React/DOM，可单测） |
| `src/client/games/mini/mini.module.css` | 跨引擎共享样式（wrap/header/btn/cell/grid/banner/...） |
| `src/client/games/mini/engines-simple.tsx` | 约 24 引擎: clicker/reaction/aim/whack/bubble/etch/pixel/number/math/trivia/hangman/scramble/typing/stroop/riddle/coin/dice/spinner/color/slots/blackjack/highlow/war/cat-clicker + `bind(Engine, props)` 包装器 |
| `src/client/games/mini/engines-strategy.tsx` | 约 18 引擎: ttt/memory/simon/lights/slide/peg/mastermind/24game/water/hanoi/connect4/knight/mines/sudoku/nonogram/2048/nqueens/pathfind |
| `src/client/games/mini/engines-arcade.tsx` | 约 10 引擎: snake/breakout/pong/flappy/dodge/dino/asteroids/maze/colorseq/catch |
| `src/client/games/mini/catalog.ts` | `MINI_CATALOG`: 192 条 `{id,name,icon,Component: bind(Engine, props)}` 静态数组 + `GROUP_RULES`/`applyGroupRule` 折叠规则 |
| `src/client/games/mini/index.ts` | `registerMiniGames(registry)`: O(N) 注册 catalog 全部条目并附加 group 元数据 |
| `tests/mini.spec.ts` | catalog 条目数 ≥100、id 唯一、折叠规则覆盖、注册/销毁完整性 |

## Commands

```sh
pnpm run typecheck    # tsc --noEmit（自身源码 0 错误；../dsh vendor 预存在错误忽略）
pnpm test             # vitest run（wordle / match3 / 注册表 / mini catalog 单元测试）
pnpm run build        # tsc + tsdown → lib/index.js, lib/invariant.js, lib/client.js
pnpm run bundle:client  # 只跑 tsdown，绕开 tsc 阶段（用于跳过 vendor 类型错误重建产物）
```

## Mini-game 引擎参数化模式

每个 mini 引擎是标准 React 组件，接收 `GameProps & EngineProps`。`bind(Engine, props)` 返回仅接收 `GameProps` 的包装组件，把 `EngineProps` 闭包进去，作为 `Game.Component` 塞进注册表。同一引擎绑定不同参数即生成不同游戏条目（如 `Snake` × `{size, speedMs}` → 5 款贪吃蛇）。

新增 mini 游戏不需要写新引擎——只要在 `catalog.ts` 加一行 `{ id, name, icon, Component: bind(SomeEngine, newProps) }`。新增引擎品类才需要往 `engines-*.tsx` 加组件 + 在 `shared.ts` 加纯逻辑。

## 菜单折叠（GROUP_RULES）

同族变体（同引擎多参数）折叠为一个菜单组：

- `GROUP_RULES`（catalog.ts 末尾）按 id 前缀/后缀匹配分组，**首条命中生效**（如 `colorseq-` 必须排在 `color-` 前）；未命中规则的游戏为独立条目。
- `registerMiniGames` 注册时把 `group` 附加到 Game 上；`D399Overlay` 的 `bucketGames` 把同组条目聚成一个可折叠组（组内单成员自动降级为普通卡片）。
- 新增引擎时同步在 `GROUP_RULES` 加规则，并跑 `tests/mini.spec.ts`（断言所有 catalog 条目都能解析到组、组 id 唯一）。
- 第三方插件注册游戏时带 `group: { id, name, icon }` 即可参与折叠。

## 生成状态检测

overlay 不走 slot 系统（是 `position: fixed` DOM 贡献，portal 到 `document.body`），直接订阅 `ctx.sessions.list`：

1. `useSyncExternalStore(list.subscribe, getSnapshot)` 订阅 list 快照。
2. `getSnapshot` 读 `byId[current].running`，做引用稳定性缓存（running 未翻转时返回同一引用）。
3. `running: false→true` 弹 teaser；`true→false` 收 teaser（菜单/游戏中除外）。

## 可拓展性

`ctx.d399Games` 是 cordis 服务。第三方插件：

```ts
export const inject = ['d399Games']
export function apply(ctx) {
  ctx.effect(() => ctx.d399Games.register({
    id: 'snake', name: '贪吃蛇', icon: '🐍', Component: Snake,
  }), 'my-plugin: snake')
}
```

## Gotchas

- overlay 是 `position: fixed`，通过 `createPortal` 挂到 `document.body`，逃逸任何 ancestor `transform` / `overflow: hidden`。
- `teaser` 每次生成周期只弹一次；用户关闭后同一次生成不再弹（生成结束重新武装）。
- `useSessionRunning` 的 snapshot 缓存必须按 `(id, running)` 双键比较，否则切换会话时会误触发。
- match3 `generateBoard` 用「试错法」避免初始匹配（最多 20 次重试），理论上极小概率仍可能有匹配——`resolveCascades` 会兜底清除。

---
> Source: [HuanLinOTO/dsh-plugin-d399](https://github.com/HuanLinOTO/dsh-plugin-d399) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
