## sts2-web

> 本文件为在此仓库中工作的编码代理提供指引。

# AGENTS.md

本文件为在此仓库中工作的编码代理提供指引。

## 硬性规则

### 任何情况下都不要影响旧存档的读取

线上玩家的存档在各自浏览器里，丢了无法恢复。任何改动都必须保证已有存档还能正常读出来，没有例外。

读档**没有迁移这一层**：`packages/core/src/overrides.ts` 把 `MigrationManager.LoadWithAggressiveRecovery` 换成了直接反序列化，所以存档格式一变，旧存档就读不出来，没有兜底。具体来说：

- 不要改名、删除存档 DTO 的 JSON 成员，也不要改它们的类型或含义；新增成员必须在旧存档缺这个字段时也能读。
- 不要改存档里引用的模型 Id、存档文件的路径和文件名（`user://` 下）。
- 不要改存储位置：IndexedDB 的 `sts2fs` 库、localStorage 的 `sts2fs:` 前缀（IndexedDB 不可用时的退路）和 `sts2fs-j:` 日志（页面关闭时的写入，下次 `vfs.mount()` 重放），都在 `packages/core/src/rt/godot.ts`。
- 更早版本存在 localStorage 的存档，首次启动时迁移到 IndexedDB 的逻辑必须一直可用。
- 重新生成规则层（`pnpm gen`）或升级游戏版本可能改动存档 DTO，合入前要确认旧存档仍能读取。升级游戏版本按 `.claude/skills/upgrading-game-version/SKILL.md` 的阶段走。
- 确实需要改格式时，先实现兼容旧格式的读取路径，并用旧存档做测试；拿不准就停下来问。

验证：`pnpm -F @sts2/core test`（`test/save.test.ts` 是序列化往返；`test/old-saves.test.ts` 读 `test/fixtures/saves-v*/` 里从真实浏览器导出的旧存档，这些样本不要修改或重新生成），以及 `tools/e2e/continue.mjs`（保存并退出 → 刷新 → 继续）。

### 不要手改生成的代码

`packages/core/src/gen/` 由转译器生成。行为和原作不一致时，在 `tools/cs2ts`（转译器）、`packages/core/src/rt`（运行时）或 `packages/core/src/overrides.ts`（少量手写替换）里修，不要直接编辑 `gen/`。

### 使用范围

非官方学习项目，仅供学习使用。`assets/` 和 `packages/core/src/gen/` 来自《Slay the Spire 2》，版权归 Mega Crit 所有。

## 常用命令

需要 Node 20+ 和 pnpm。游戏资源和生成代码已在仓库里，不需要安装游戏。

```bash
pnpm install
pnpm dev                         # 开发服务器 http://127.0.0.1:47173/
pnpm build                       # tsc + vite 构建，产物在 packages/app/dist
pnpm -F @sts2/app preview        # 预览构建 http://127.0.0.1:47174/
pnpm -F @sts2/core test          # 无头测试（Vitest，约 2 分钟）
pnpm -F @sts2/core exec vitest run test/save.test.ts          # 单个测试文件
pnpm -F @sts2/core exec vitest run test/save.test.ts -t "<用例名>"   # 单个用例
pnpm -F @sts2/app exec tsc -p . --noEmit                       # 只做应用的类型检查
pnpm gen                         # 重新转译规则层（需要 ref/decompiled 和 .NET 9，见 docs/development.md）
node tools/check-refs.mjs        # 手写代码引用的规则层类 / 重载 / 模型 Id / 本地化键是否还在（`G` 是 any，tsc 查不出）
```

页面参数：`?seed=<种子>`、`?lang=zhs`、`?unlock=all`、`?tutorials=off`、`?scene=<场景路径>`（仅开发服务器）。原版的开发者控制台按 `` ` `` 开关，不需要参数，命令见 `docs/dev-console.md`。

### 浏览器端测试（`tools/e2e`）

Playwright 驱动真实界面，默认连 `http://127.0.0.1:47173/`（`URL=` 可改），需要用 `CHROME=` 指向已安装的 headless shell（`npx playwright install chromium-headless-shell`）。

```bash
CHROME=<路径> FAST=1 GOD=1 node tools/e2e/play.mjs /tmp/play     # 自动游玩；SEED / CHAR / FULL 等开关见文件头注释
CHROME=<路径> node tools/e2e/coverage.mjs /tmp/cov               # 卡牌 / 药水 / 遭遇战 / 遗物 / 事件覆盖
CHROME=<路径> node tools/e2e/continue.mjs /tmp/cont              # 保存并退出 → 刷新 → 继续
```

对着开发服务器跑的时候不要同时改 `packages/`：HMR 会在运行中途重载模块，产生假失败。长时间的运行改用构建产物。其余脚本见 `docs/development.md`。

## 架构

规则层不是手写的，而是把原作（v0.98.3）反编译出的 C# 用 Roslyn 转译器整体转成 TypeScript；手写部分是让它跑起来的运行时，以及接在它下面的表现层。

```
游戏安装包
  ├─ tools/extract.py、audio.py、scenes.py …  ──▶  assets/                  美术、动画、音频、文本、场景
  └─ tools/decompile.sh  ──▶  ref/decompiled
                                 └─ tools/cs2ts（Roslyn） ──▶  packages/core/src/gen   转译出的规则层

packages/core   运行时：C#/.NET 语义（BCL、集合、LINQ、Task、JSON）与 Godot API 的 TypeScript 实现
packages/app    表现层：Preact UI + Pixi 渲染，通过 bridge 接上规则层调用的场景节点
```

- **`packages/core/src/gen`**：`sts2.ts` 是规则层，`stubs.ts` 是 Godot 节点等外部类型的桩。外部类型统一走 `$.ext("全名")`。
- **`packages/core/src/rt`**：手写运行时。`godot.ts` 里的 vfs 承载 `user://`（存档），启动时 `vfs.mount()` 把 IndexedDB 的文件全部读入内存，因为规则层的文件读写是同步的；之后的写入异步落盘。
- **`packages/core/src/shell.ts`**：`NGame` 的非界面部分（初始化、开新局、继续 / 放弃存档）。
- **`packages/app/src/bridge.ts`**：用 TypeScript 实现规则层会调用的 Godot 节点单例（`NRun`、`NCombatRoom`、`NMapScreen`、商店、奖励和选牌界面等）。视图对象改完字段后调 `invalidate()`，由 `store.ts` 的 rAF 循环重新渲染。
- **`packages/app/src/ui`**：Preact 界面，按 1920 × 1080 逻辑坐标照原作场景摆放。**`packages/app/src/render`**：Pixi 8 + spine-pixi（生物、场景、粒子、着色器）。
- **两种模式**：测试模式（`TestMode.IsOn`，无动画无等待，Vitest 用）和浏览器模式（走原作的非测试分支，由 bridge 提供节点）。

写桥接和界面代码时：

- 以原作为准。注释里的 `NMerchantRoom`、`NOverlayStack` 这类名字是原作的 Godot 节点，对应的 C# 在 `ref/decompiled`（不入库，需按 `docs/development.md` 重新生成）。
- C# 的扩展方法在生成代码里是静态方法：写 `G.GodotTreeExtensions.AddChildSafely(parent, child)`，不要写 `parent.AddChildSafely(child)`。
- 规则层的委托按订阅顺序触发，原作里先订阅的回调（例如商店库存的 `UpdateEntries`）会先于 bridge 的回调执行，依赖顺序的状态要按原作节点的做法处理。

`ref/` 和 `_work/` 不入库；`assets/` 和 `packages/core/src/gen/` 入库。更完整的设计与实施记录见 `docs/sts2-web-port-plan.md`。

## 提交

提交信息用 Conventional Commits，scope 沿用现有的 `app` / `core` / `tools` / `assets`，例如 `fix(app): …`。

---
> Source: [moonrailgun/sts2-web](https://github.com/moonrailgun/sts2-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
