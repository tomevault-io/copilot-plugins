## bit-framework

> > 基于 Cocos Creator 3.x 的游戏框架 Monorepo，11 个独立 TypeScript 模块，pnpm workspace 管理，Rollup 构建。

# Bit Framework — AI 开发指令

> 基于 Cocos Creator 3.x 的游戏框架 Monorepo，11 个独立 TypeScript 模块，pnpm workspace 管理，Rollup 构建。

## 目录结构

```
bit-framework/
├── bit-core/         # 核心工具（Time, Platform, Timer, Utils）
├── bit-ui/           # FairyGUI UI 管理（窗口、装饰器）
├── bit-ecs/          # 高性能 ECS 架构
├── bit-ec/           # Cocos 适配 EC 架构
├── bit-event/        # 全局事件系统
├── bit-net/          # HTTP + WebSocket
├── bit-assets/       # 资源加载管理
├── bit-quadtree/     # 四叉树碰撞检测
├── bit-behaviortree/ # AI 行为树
├── bit-condition/    # UI 条件显示（红点）
├── bit-minigame/     # 小游戏平台适配
├── bit-hotupdate/    # 热更新系统
├── docs/             # 架构和命令文档
└── rollup.config.base.mjs  # 所有模块共用的构建配置
```

## 关键文件索引

| 任务 | 位置 |
|------|------|
| 修改构建配置 | `rollup.config.base.mjs` |
| 添加 pnpm 脚本 | 根 `package.json` |
| 模块依赖关系 | `docs/ARCHITECTURE.md` |
| 构建/发布命令 | `docs/COMMANDS.md` |
| 各模块入口 | `bit-xxx/src/index.ts` |
| UI 窗口基类 | `bit-ui/src/window/` |
| ECS 组件/系统 | `bit-ecs/src/` |
| 装饰器定义 | 各模块 `*Decorator.ts` |

## 模块依赖

```
独立（零依赖）: bit-event · bit-ecs · bit-net · bit-assets · bit-quadtree · bit-behaviortree
依赖 bit-core:  bit-ui · bit-condition · bit-minigame · bit-hotupdate
特殊:           bit-ec → bit-event  |  bit-hotupdate → bit-core + bit-net
```

## 约定

### 包命名
- npm scope: `@gongxh/bit-*`
- workspace 引用: `"@gongxh/bit-core": "workspace:*"`

### 构建输出（每个模块 dist/ 目录）
- `bit-xxx.mjs` / `.cjs` — ESM / CommonJS
- `bit-xxx.min.mjs` / `.min.cjs` — 压缩版（生产用）
- `bit-xxx.d.ts` — TypeScript 类型定义

### TypeScript
- `strictNullChecks: false` — Cocos Creator 兼容性
- `experimentalDecorators: true` — 装饰器广泛使用
- 每个模块独立 `tsconfig.json`，继承根配置

### 装饰器
| 模块 | 装饰器 | 用途 |
|------|--------|------|
| bit-ui | `@uiclass` `@uiprop` `@uiclick` | UI 窗口注册 |
| bit-ecs | `@ecsclass` `@ecsprop` `@ecsystem` | ECS 组件/系统 |
| bit-ec | `@ecclass` `@ecprop` | EC 组件 |
| bit-condition | `@condition` | 条件类注册 |
| bit-behaviortree | `@ClassAction` `@prop` | 行为树节点 |

## 禁止事项

- **禁止循环依赖** — 单向依赖流，双向通信用 bit-event
- **禁止 `as any`** — 类型安全优先
- **禁止空 catch** — 必须处理错误
- **禁止直接修改 dist/** — 只修改 src/，重新构建

## 开发工作流

```bash
# 1. 修改 src/ 代码
# 2. 构建验证
pnpm build:core          # 构建单模块

pnpm build:all           # 构建所有模块

# 4. 提交
git add <files>
git commit -m "type(scope): description"
```

### commit 类型
- `feat` — 新功能
- `fix` — bug 修复
- `refactor` — 重构
- `docs` — 文档
- `chore` — 构建/版本/配置

## Skills（可用的 AI 命令）

| 命令 | 用途 |
|------|------|
| `/release` | 完整发版：升版本 → 构建 → 提交 → 发布 npm → 打 tag |
| `/changelog` | 根据 git log 生成 CHANGELOG 条目 |

## 参考文档

- [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) — 架构设计、模块分层、设计原则
- [docs/COMMANDS.md](./docs/COMMANDS.md) — 所有构建/发布命令速查

---
> Source: [gongxh0901/bit-framework](https://github.com/gongxh0901/bit-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
