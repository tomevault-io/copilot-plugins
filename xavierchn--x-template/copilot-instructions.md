## x-template

> 这是一个 DOTA2自定义游戏（DOTA2RPG）制作项目模板，关于游戏的具体设定，请读取 GAME_DESIGN.MD 文件了解游戏的整体设计内容和用户要求。

# AGENTS.md

## 项目概述

这是一个 DOTA2自定义游戏（DOTA2RPG）制作项目模板，关于游戏的具体设定，请读取 GAME_DESIGN.MD 文件了解游戏的整体设计内容和用户要求。

本项目使用 **yarn** 作为包管理器，请勿使用 npm 安装依赖。

## 关键路径

### 游戏逻辑 (`game/`)

| 路径 | 用途 |
|---|---|
| `game/addoninfo.txt` | Dota 2 Addon 元信息（地图名、玩家数等） |
| `game/maps/` | 地图文件（`.vpk` 编译产物） |
| `game/resource/addon.csv` | 本地化源文件（CSV，Tokens/English/SChinese） |
| `game/resource/kv_generated.csv` | 从 Excel `#Loc{}` 标签自动生成的本地化，请勿修改此文件 |
| `game/scripts/custom_net_tables.txt` | 自定义网络表声明（客户端-服务端数据同步） |
| `game/scripts/tsconfig.json` | TSTL 开发环境编译配置 |
| `game/scripts/tsconfig.prod.json` | TSTL 生产环境编译配置（加密） |

### TSTL 源码 (`game/scripts/src/`)

| 路径 | 用途 |
|---|---|
| `src/addon_game_mode.ts` | 服务端入口（游戏模式初始化） |
| `src/addon_game_mode_client.ts` | 客户端入口 |
| `src/abilities/` | 技能源码（`@registerAbility()` 装饰器注册），技能自己的 modifier 也放在同一文件中 |
| `src/modifiers/` | 独立的机制性 modifier（如 `modifier_generic_arc` 抛物线运动，使用 `@registerModifier()` 装饰器注册），仅放置不被技能引用的通用 modifier |

**Modifier 放置规则**：
- 技能/物品自己的 modifier → 直接放在技能/物品的同一个 `.ts` 文件中
- 独立的机制性 modifier（不被任何技能/物品引用，如通用抛物线运动、通用buff等）→ 单独放在 `src/modifiers/` 目录下
| `src/modules/` | 游戏模块：`GameConfig.ts`（全局配置）、`Debug.ts`（调试指令）、`index.ts`（模块激活）等等 |
| `src/server/core/` | OpenAPI 自动生成的服务端请求客户端（`request.ts`、`OpenAPI.ts`） |
| `src/utils/dota_ts_adapter.ts` | 核心适配器：`BaseAbility`、`BaseModifier`、`registerAbility`、`registerModifier` |
| `src/utils/tween.ts` | 补间动画工具 |
| `src/utils/timer_utils.ts` | 定时器工具 |
| `src/utils/tstl-utils.ts` | TSTL 辅助函数 |
| `src/utils/precache.ts` | 预缓存工具 |
| `src/utils/xnet-table/` | 自定义网络表读写封装 |
| `src/utils/performance/` | 火焰图性能分析器（`flame_graph_profiler.ts`） |
| `src/utils/testing/` | 测试框架（`test_framework.ts`） |
| `src/utils/libs/` | 加密库（AES、Base64、Deflate） |
| `src/utils/json.lua` | JSON 解析库（Lua） |
| `src/utils/timers.lua` | 定时器库（Lua） |
| `src/utils/popup.lua` | 弹窗工具（Lua） |

### KV 配置 (`game/scripts/npc/`)
**注意：编写了KV文件后，必须在各自的入口文件中引用。**
| 路径 | 用途 |
|---|---|
| `npc/npc_abilities_custom.txt` | 技能入口文件 |
| `npc/abilities.txt` | 技能 KV 定义（从 Excel 生成，请不要修改此文件） |
| `npc/abilities/*.txt` | 你生成的技能文件请写在此目录下，文件名与技能名一致 |
| `npc/npc_heroes_custom.txt` | 英雄入口文件（`#base "heroes.txt"`） |
| `npc/heroes.txt` | 英雄 KV 定义（覆盖原版英雄属性、绑定技能，由 Excel 生成，请不要修改此文件） |
| `npc/herolist.txt` | 可选英雄列表（启用/禁用） |
| `npc/npc_items_custom.txt` | 物品入口文件（`#base "items_list.txt"`） |
| `npc/items_list.txt` | 物品 KV 定义（从 Excel 生成，请不要修改此文件） |
| `npc/items_list/*.txt` | 你生成的物品文件请写在此目录下，文件名与物品名一致 |
| `npc/npc_units_custom.txt` | 单位入口文件（`#base "custom_units.txt"`） |
| `npc/custom_units.txt` | 自定义单位 KV 定义（生物、怪物等），请勿修改此文件 |
| `npc/custom_units/*.txt` | 你生成的单位文件请写在此目录下，文件名与单位名一致 |
| `npc/round_settings.txt` | 刷怪回合配置（从 `刷怪表.xlsx` 生成） |

### 编译输出 (`game/scripts/vscripts/`)

| 路径 | 用途 |
|---|---|
| `vscripts/` | TSTL 编译输出目录（`.lua` 文件），**勿编辑**，已 gitignore |

### UI 前端 (`content/panorama/`)

| 路径 | 用途 |
|---|---|
| `panorama/tsconfig.json` | Panorama TypeScript 配置 |
| `panorama/webpack.dev.js` | Webpack 开发配置 |
| `panorama/webpack.prod.js` | Webpack 生产配置 |
| `panorama/images/` | UI 图片资源（`.psd`、`.png`、`.jpg`） |
| `panorama/src/hud/` | 主 HUD 界面（`layout.xml` + `script.tsx` + `styles.less`） |
| `panorama/src/loading-screen/` | 加载界面 |
| `panorama/src/end_screen/` | 结算界面 |
| `panorama/src/hooks/` | React Hooks：`useXNetTable`、`useKeyboard`、`useTimer`、`useInterval` 等 |
| `panorama/src/def/` | 类型定义：`defs.ts`（全局类型）、`local_event_def.ts`（本地事件） |
| `panorama/src/utils/event-bus.ts` | 事件总线 |
| `panorama/src/utils/x-nettable-dispatcher.ts` | 网络表数据分发器 |
| `panorama/src/utils/draggable_window/` | 可拖拽窗口组件 |
| `panorama/src/utils/flame_graph/` | 火焰图 UI 组件 |
| `panorama/src/utils/react-panorama-qrcode/` | 二维码组件 |
| `panorama/src/utils/keybinding.ts` | 键位绑定工具 |
| `panorama/src/utils/sequential-actions.ts` | 顺序动作队列 |
| `panorama/src/test/` | 测试文件 |

### 地图 (`content/maps/`)

| 路径 | 用途 |
|---|---|
| `content/maps/temp.vmap` | 地图源文件（Hammer 编辑器格式，请勿编辑） |

### 共享类型 (`shared/`)

此文件夹主要用来在 game/scripts/src 和 content/panorama/src 中共享类型定义，避免 GameEvents和CustomNetTables等API报错。

| 路径 | 用途 |
|---|---|
| `shared/gameevents.d.ts` | 游戏事件类型定义（`GameEvents` 发送/接收） |
| `shared/net_tables.d.ts` | 网络表类型定义（`CustomNetTables`） |
| `shared/x-net-table.d.ts` | 自定义网络表类型（`XNetTable`） |

### Excel 数据表 (`excels/`)

| 路径 | 用途 |
|---|---|
| `excels/技能表.xlsx` | 技能数据（→ `abilities.txt`） |
| `excels/英雄表.xlsx` | 英雄数据（→ `heroes.txt`） |
| `excels/英雄列表.xlsx` | 英雄列表（→ `herolist.txt`） |
| `excels/单位表.xlsx` | 单位数据（→ `custom_units.txt`） |
| `excels/物品表.xlsx` | 物品数据（→ `items_list.txt`） |
| `excels/刷怪表.xlsx` | 刷怪配置（→ `round_settings.txt`） |

### 根目录配置文件

| 路径 | 用途 |
|---|---|
| `package.json` | 依赖与脚本命令 |
| `gulpfile.ts` | Gulp 构建任务（Excel→KV、KV→JSON、CSV→本地化、图片预缓存、LESS 编译） |
| `tsconfig.json` | 根 TypeScript 配置 |

## 核心规则

- **禁止未经用户同意执行 commit**：任何 git commit 操作必须获得用户明确许可后方可执行。
- **commit 前必须执行 `yarn lint`**：确保所有 lint 检查通过后再提交。

## 测试框架

测试框架文档详见 `game/scripts/src/utils/testing/readme.md`。

### 编写测试用例

```typescript
// game/scripts/src/utils/testing/my_test.ts
import { describe, it, expect, delay } from './test_framework';

describe('MyModule', () => {
    it('同步测试', () => {
        expect(1 + 1).toBe(2);
    });

    it('异步测试', async () => {
        let flag = false;
        Timers.CreateTimer(0.1, () => { flag = true; return null; });
        await delay(0.2);
        expect(flag).toBeTruthy();
    });
});
```

### 注册测试（必须！）

测试文件编写后，必须在 `game/scripts/src/utils/testing/index.ts` 中导入才能被注册：

```typescript
// testing/index.ts 末尾
import './my_test';
```

**不要在 `dev_commands.ts` 或其他文件中导入测试文件**，必须通过 `testing/index.ts` 注册。

### 引用测试框架 API

- 在 `game/scripts/src/utils/testing/` 下的文件：`import { ... } from './test_framework'`
- 在 `dev_commands.ts` 等外部文件：`import { runAll, printResult } from '..'`（即 `testing/index.ts`，而非直接 `../test_framework`）

**规则**：凡是要使用 `runAll` / `printResult`，一律从 `testing/index.ts` 导入（`'..'` 或 `'../utils/testing'`），不要直接从 `test_framework.ts` 导入，否则测试文件不会被注册。

### 运行测试

在游戏内聊天框输入（需 tools mode，Dev 指令）：
- `-tx` — 运行所有测试
- `-tx Timer` — 运行名称包含 "Timer" 的套件（大小写不敏感的子串匹配）

### 已知陷阱

| 问题 | 说明 | 正确做法 |
|------|------|---------|
| 测试文件未注册 | 忘记在 `testing/index.ts` 添加 `import './my_test'` | 在 `index.ts` 中添加 side-effect import |
| 绕过 `index.ts` 导入 | 直接 `from '../test_framework'` 导致测试文件不加载 | 必须 `from '..'` 或 `from '../utils/testing'` |
| 测试结果 0/0 | 通常是上述两种原因导致套件未注册 | 检查 `testing/index.ts` 的导入 |

## 错误检查

在修改代码后，使用以下命令分别检查前后端报错，不要使用 `tsc` 直接检查：

- **后端（TSTL 脚本）**：`npx tstl --project game/scripts/tsconfig.json`
- **前端（Panorama UI）**：`npx tsc --project content/panorama/tsconfig.json --noEmit`
- **Lint/格式化**：`yarn lint`

---
> Source: [XavierCHN/x-template](https://github.com/XavierCHN/x-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
