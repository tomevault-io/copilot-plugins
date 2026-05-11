## vue3-project-structure

> Easy Window 是一款基于 Vue3 + Tauri 的桌面端UI生成工具，采用现代化前端技术栈。

# Vue3 项目架构和文件组织规范

## 项目概述
Easy Window 是一款基于 Vue3 + Tauri 的桌面端UI生成工具，采用现代化前端技术栈。

## 核心技术栈
- Vue 3 (Composition API)
- TypeScript 
- Element Plus UI组件库
- Pinia 状态管理
- Vue Router 路由管理
- Tauri 2.x 桌面应用框架
- Vite 构建工具

## 目录结构规范

### 源码目录 `src/`
```
src/
├── main.ts                    # 应用入口文件
├── App.vue                    # 根组件
├── style.css                  # 全局样式
├── components/                # 组件目录
│   ├── library/              # 组件库
│   │   ├── basic/           # 基础组件
│   │   └── custom/          # 自定义组件
│   └── PropertyPanelEditors/ # 属性编辑器
├── views/                     # 页面组件
├── router/                    # 路由配置
├── stores/                    # Pinia状态管理
├── composables/               # 组合函数
├── types/                     # TypeScript类型定义
├── utils/                     # 工具函数
├── services/                  # API服务
├── plugins/                   # 插件配置
├── directives/                # 自定义指令
├── mixins/                    # 混入
├── config/                    # 配置文件
├── templates/                 # 模板文件
├── examples/                  # 示例代码
└── assets/                    # 静态资源
```

## 开发规范

### 文件命名
- 组件文件: PascalCase (如 `ComponentName.vue`)
- 工具函数: camelCase (如 `utilityFunction.ts`)
- 配置文件: kebab-case (如 `api-config.ts`)

### 组件开发规范
- 优先使用 Composition API
- 使用 `<script setup>` 语法
- 采用 TypeScript 进行类型约束
- 组件应具备良好的类型定义和注释

### 状态管理
- 使用 Pinia 进行状态管理
- Store 文件放在 `stores/` 目录
- 每个模块对应一个独立的 store

### 样式规范
- 优先使用 Element Plus 组件
- 减少自定义 CSS 样式编写
- 遵循响应式设计原则
- 使用 CSS 变量进行主题配置

---
> Source: [suyancc/easy_window](https://github.com/suyancc/easy_window) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
