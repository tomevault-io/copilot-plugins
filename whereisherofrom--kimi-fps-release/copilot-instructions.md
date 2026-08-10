## kimi-fps-release

> FPS 网页游戏（Valorant TPS 风格），基于 React + Three.js + @react-three/fiber 构建的 3D 第一人称射击游戏。

# 项目上下文

## 项目概述

FPS 网页游戏（Valorant TPS 风格），基于 React + Three.js + @react-three/fiber 构建的 3D 第一人称射击游戏。

## 技术栈

- **核心**: Vite 7, TypeScript, Express, React 19
- **3D 渲染**: Three.js, @react-three/fiber, @react-three/drei, @react-three/postprocessing
- **UI**: Tailwind CSS, shadcn/ui (Radix UI), Lucide Icons
- **状态管理**: Zustand
- **路由**: react-router (HashRouter)
- **动画**: GSAP, Lenis

## 目录结构

```
├── scripts/            # 构建与启动脚本
│   ├── build.sh        # 构建脚本
│   ├── dev.sh          # 开发环境启动脚本
│   ├── prepare.sh      # 预处理脚本
│   └── start.sh        # 生产环境启动脚本
├── server/             # 服务端逻辑
│   ├── routes/         # API 路由
│   ├── server.ts       # Express 服务入口
│   └── vite.ts         # Vite 中间件集成
├── src/                # 前端源码
│   ├── components/
│   │   ├── game/       # 游戏核心组件 (CameraController, Player, Scene, Weapon, TargetSystem, Effects)
│   │   └── ui/         # UI 组件 (HUD, Crosshair, TopBar, TutorialOverlay, shadcn/ui)
│   ├── hooks/          # 自定义 hooks (useGameControls, usePointerLock)
│   ├── pages/          # 页面 (GamePage)
│   ├── stores/         # Zustand 状态 (gameStore)
│   ├── utils/          # 常量与工具
│   ├── config.ts       # 游戏配置
│   ├── App.tsx         # 根组件
│   ├── main.tsx        # 客户端入口
│   └── index.css       # 全局样式
├── public/             # 静态资源
│   ├── models/         # 3D 模型 (ak47.glb)
│   ├── audio/          # 音效
│   └── images/         # 图片资源
├── index.html          # 入口 HTML
├── package.json        # 项目依赖管理
├── tsconfig.json       # TypeScript 配置
├── tailwind.config.cjs # Tailwind 配置 (CJS 格式)
└── vite.config.ts      # Vite 配置
```

## 包管理规范

**仅允许使用 pnpm** 作为包管理器，**严禁使用 npm 或 yarn**。

## 开发规范

- 使用 Tailwind CSS 进行样式开发
- 路径别名: `@/` 映射到 `./src/`
- 游戏核心逻辑在 `src/components/game/` 目录
- 游戏状态通过 Zustand (`src/stores/gameStore.ts`) 管理

### 编码规范

- 默认按 TypeScript `strict` 心智写代码；优先复用当前作用域已声明的变量、函数、类型和导入，禁止引用未声明标识符或拼错变量名。
- 禁止隐式 `any` 和 `as any`；函数参数、返回值、解构项、事件对象、Express `req`/`res`、`catch` 错误在使用前应有明确类型或先完成类型收窄，并清理未使用的变量和导入。

---
> Source: [WhereIsHeroFrom/kimi_fps_release](https://github.com/WhereIsHeroFrom/kimi_fps_release) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
