## dabase-tool

> 这是一个全栈应用程序，旨在从 DM8（达梦）数据库中导出数据库模式 (DDL) 和数据。它采用基于 Rust 的高性能后端（通过 ODBC 进行数据库操作）和现代化的 React 前端。

# DM8 数据库导出工具 (DM8 Database Export Tool)

## 项目概览
这是一个全栈应用程序，旨在从 DM8（达梦）数据库中导出数据库模式 (DDL) 和数据。它采用基于 Rust 的高性能后端（通过 ODBC 进行数据库操作）和现代化的 React 前端。

**最新更新：** 前端已全面升级为“科技风”暗黑主题，集成了动态粒子背景和流畅的交互动画，提供沉浸式的用户体验。

## 设计思路 / 运行模式
- **分层职责**：前端专注 UI + API 调用，后端专注 ODBC/导出逻辑；接口清晰，便于后续封装为桌面应用。
- **驱动内置优先**：仓库附带 DM8 ODBC 驱动及依赖（`drivers/dm8`），后端连接串优先使用内置或 `DM8_DRIVER_PATH`，不足时再回落系统驱动。
- **运行方式**：
  - **HTTP 开发**：启动后端 (3000) 时注入 `LD_LIBRARY_PATH=$(pwd)/../drivers/dm8` 和 `DM8_DRIVER_PATH=$(pwd)/../drivers/dm8/libdodbc.so`，前端 Vite Dev (5173/5174) 直连 `/api`。
  - **桌面封装（规划中）**：Tauri 打包 AppImage/Windows exe，随包携带驱动并在启动时自动设置库搜索路径。

## 技术栈

### 后端 (Backend)
*   **语言:** Rust
*   **Web 框架:** Axum
*   **数据库访问:** `odbc-api` (通过 DM8 ODBC 驱动)
*   **日志:** `tracing`
*   **序列化:** `serde`, `serde_json`

### 前端 (Frontend)
*   **核心框架:** React (Vite 构建)
*   **语言:** TypeScript
*   **UI 组件库:** Ant Design (`antd`) - **定制暗黑主题**
*   **路由管理:** React Router (`react-router-dom`)
*   **状态管理:** Zustand (`zustand`)
*   **数据请求:** React Query (`@tanstack/react-query`), Axios
*   **动画引擎:** Anime.js (`animejs`) - 用于复杂的过渡和背景动画

## 项目结构

```
tool-database/
├── backend/                # Rust API 服务器
│   ├── src/
│   │   ├── api/            # API 路由处理
│   │   ├── db/             # 数据库适配器
│   │   ├── export/         # 导出逻辑
│   │   └── main.rs         # 入口点
│   └── ...
├── frontend/               # React 用户界面
│   ├── src/
│   │   ├── assets/         # 静态资源
│   │   ├── components/     # UI 组件
│   │   │   ├── TechBackground.tsx  # [新增] 科技感粒子背景
│   │   │   └── ...
│   │   ├── layouts/        # [新增] 布局组件 (MainLayout)
│   │   ├── pages/          # [新增] 页面组件 (ExportWizard)
│   │   ├── router/         # [新增] 路由配置
│   │   ├── services/       # API 客户端
│   │   ├── store/          # [新增] Zustand 状态管理 (useExportStore)
│   │   ├── types/          # 类型定义
│   │   └── App.tsx         # 全局配置 (Theme, Providers)
│   └── ...
└── README.md
```

## UI/UX 设计系统 (Design System)

本项目采用了**暗黑科技风格 (Dark Tech Style)**，主要设计特征如下：

1.  **色彩体系**:
    *   **主题模式**: Ant Design Dark Algorithm。
    *   **主色调**: Cyber Green (`#00b96b`)，用于按钮、高亮和关键文本，营造黑客/控制台氛围。
    *   **背景色**: 深邃蓝黑渐变 (`#001529` -> `#000000`)。

2.  **视觉效果**:
    *   **动态背景**: 自定义 Canvas 组件 (`TechBackground`)，绘制动态网格和连接粒子，模拟数据流和神经网络。
    *   **玻璃拟态**: 卡片和容器采用半透明背景 (`rgba`) 配合背景模糊 (`backdrop-filter`)，增强层次感。
    *   **锐利边角**: 圆角半径 (`borderRadius`) 设置为 2px，强化硬朗的科技感。

3.  **交互动画**:
    *   使用 `anime.js` 实现页面入场和步骤切换的平滑过渡（淡入、位移、缩放）。
    *   操作反馈迅速，配合 Ant Design 的 Message 组件。

## 快速开始

### 后端
```bash
cd backend
# 确保已配置 .env
cargo run
```

### 前端
```bash
cd frontend
npm install
npm run dev
```
访问 `http://localhost:5173` 体验新界面。

## 关键文件与逻辑

*   **`frontend/src/store/useExportStore.ts`**: 使用 Zustand 集中管理应用状态（连接信息、选中表、向导步骤），替代了复杂的 Props 传递。
*   **`frontend/src/components/TechBackground.tsx`**: 核心视觉组件，包含 Canvas 绘图逻辑，负责渲染粒子动画。
*   **`frontend/src/pages/ExportWizard.tsx`**: 主要业务页面，通过 `anime.js` 管理步骤切换时的 DOM 动画。
*   **`frontend/src/App.tsx`**: 配置了 `ConfigProvider` 的 `theme` 属性，定义了全局暗黑模式和 Token 覆盖。

## 开发规范
*   **路径别名**: 使用 `@/` 指向 `src/` 目录。
*   **Linting**: 项目集成了 ESLint (Flat Config) 和 Prettier，提交代码前请运行 `npm run lint`。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Amerone)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Amerone)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
