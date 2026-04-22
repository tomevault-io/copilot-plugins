## urdf-studio

> > 最后更新：2026-04-15 | 技术栈：React 19.2 + TypeScript 5.8 + Three.js/R3F + Vite 6.2 + Tailwind CSS 4.1 + Zustand 5

# URDF Studio Agent Guide

> 最后更新：2026-04-15 | 技术栈：React 19.2 + TypeScript 5.8 + Three.js/R3F + Vite 6.2 + Tailwind CSS 4.1 + Zustand 5
> 完整文档索引：[docs/CATALOG.md](docs/CATALOG.md)

URDF Studio 是机器人设计、装配、可视化与导出工作台。核心能力：单模式 Editor 编辑、多 URDF 组装与桥接关节、多格式导入导出（URDF / MJCF / SDF / USD / Xacro / ZIP / .usp）、AI 生成与审阅、PDF/CSV 报告、可复用 react-robot-canvas 画布封装。

## src/ 目录结构

```
src/
├── app/            应用编排层：App shell、viewer 组合、导入导出、workspace/source sync、USD hydration
├── features/       业务功能模块
│   ├── ai-assistant/     AI 生成与审阅
│   ├── assembly/         桥接组件创建与组装
│   ├── code-editor/      源码编辑器
│   ├── editor/           Editor 统一公开入口
│   ├── file-io/          底层文件能力（格式检测、project archive、USD/SDF export、弹层）
│   ├── hardware-config/  硬件/电机配置（兼容层 re-export）
│   ├── property-editor/  属性编辑、几何编辑、碰撞优化
│   ├── robot-tree/       文件树与结构树（tree-editor/ + tree-node/）
│   └── urdf-viewer/      Editor 实现：拓扑/几何/碰撞/测量 + USD runtime + workers
├── store/          Zustand 状态层
├── shared/         共享组件、3D 基础设施、hooks、i18n、数据、调试桥接、workers
├── core/           纯逻辑：解析器、robot core、mesh loaders、parse workers、runtime diagnostics
├── lib/            对外复用的 RobotCanvas 封装（仅收稳定通用能力）
├── styles/         全局样式与语义 token
└── types/          跨模块类型定义
```

补充目录：`docs/`（Agent 上下文）、`scripts/`（回归与辅助脚本）、`packages/react-robot-canvas/`（对外发布包）、`public/usd/bindings/`（USD WASM）、`output/`（导出结果）、`tmp/`（临时验证产物）、`test/`（大型 fixture 与回归样本）

## 架构红线

```text
app -> features -> store -> shared -> core -> types
```

- 不新增反向依赖；features 之间通过 store 通信
- core/ 保持纯函数，不引入 React / UI / Feature 依赖
- 使用 `@/` 指向 `src/`
- src/lib/ 只收稳定通用能力，应用内部不当业务逻辑 source of truth
- 当前存量例外见 [architecture.md](docs/architecture.md) §3，禁止扩散

设计哲学：debuggability first（优先暴露错误，不吞错）；Linux 哲学（简单直接的数据流，不为理论优雅引入抽象层，优先通过更好的数据结构消灭特殊情况）。详见 [architecture.md](docs/architecture.md) §7-8。

## Editor 单模式

Editor 统一承载三个子域：

| 子域 | 典型任务 |
|------|---------|
| 拓扑 | Link / Joint 增删、拓扑编辑、关节参数 |
| 几何 / 碰撞 / 测量 | Visual / Collision、mesh、材质、碰撞变换 |
| 硬件配置 | 电机型号、传动比、阻尼、摩擦 |

公开入口 `features/editor/index.ts`，实现位于 `features/urdf-viewer/`，跨子域交互在 `app/` 或 `shared/components/3d/`。详见 [viewer.md](docs/viewer.md)。

## 状态管理

| Store | 职责 |
|-------|------|
| `robotStore` | 模型 CRUD、Undo/Redo、派生计算、闭环约束 |
| `uiStore` | 主题、语言、侧栏、面板、显示选项（含持久化） |
| `selectionStore` | 选中、悬停、pulse、focus |
| `assetsStore` | mesh、texture、robot files、motor library、USD snapshot、export cache |
| `assemblyStore` | 多 URDF 组装、BridgeJoint、组件管理、组装历史 |
| `assemblySelectionStore` | workspace 组件 / bridge / source file 选区 |
| `collisionTransformStore` | 碰撞 gizmo 瞬时 pending transform |
| `jointInteractionPreviewStore` | 跨 viewer 关节交互预览 |

约束：跨 store 协调优先放 `app/hooks/*`；USD 中间态优先落在 `assetsStore` 或 `app/utils/*`。

## 执行准则

- 优先复用现有 hooks/utils/components，不重复造轮子
- 保持类型完整性，避免 `any`
- 涉及 3D/USD/mesh 时检查材质缓存、资源释放、hydration/export 生命周期
- 新增 `ResizeObserver`、timer、worker listener、THREE 资源时必须对称 cleanup
- 单元测试邻近源码放置（`src/**/*.test.*`）

## 常用命令

```bash
npm run dev            # 开发
npm run lint           # 代码检查
npm run typecheck      # 类型检查
npm run test           # 测试
npm run build          # 构建
npm run verify:fast    # 快速验证
npm run verify:full    # 完整验证
```

## 文档导航

| 任务 | 文档 |
|------|------|
| Editor / 3D / Viewer / USD runtime | [docs/viewer.md](docs/viewer.md) |
| 导入导出 / Workspace / 组装 | [docs/file-io.md](docs/file-io.md) |
| UI 样式 / 颜色 / 主题 / 可访问性 | [docs/style-guide.md](docs/style-guide.md) |
| AI 助手 / 审阅 / skill 路由 | [docs/ai-features.md](docs/ai-features.md) |
| 架构边界 / 依赖方向 / 例外 / 设计哲学 | [docs/architecture.md](docs/architecture.md) |
| 验收清单 / 测试样本 / 回归命令 | [docs/update-rules.md](docs/update-rules.md) |
| react-robot-canvas 对外库 | [docs/robot-canvas-lib.md](docs/robot-canvas-lib.md) |
| USD runtime fallback 审计 | [docs/runtime-fallback-audit.md](docs/runtime-fallback-audit.md) |
| 完整文档索引 | [docs/CATALOG.md](docs/CATALOG.md) |

---
> Source: [OpenLegged/URDF-Studio](https://github.com/OpenLegged/URDF-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
