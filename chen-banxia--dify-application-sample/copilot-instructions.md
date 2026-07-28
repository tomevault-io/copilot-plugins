## dify-application-sample

> 这是一个基于Vue 3和Element Plus的响应式Web应用，用于展示和管理Dify智能体。提供精美的卡片式界面，支持搜索筛选和主题切换功能。

# 我的AI工坊 - Dify智能体展示平台

## 项目简介
这是一个基于Vue 3和Element Plus的响应式Web应用，用于展示和管理Dify智能体。提供精美的卡片式界面，支持搜索筛选和主题切换功能。

## 技术栈
- Vue 3 (Composition API + `<script setup>`)
- Element Plus UI组件库
- Pinia 状态管理
- Vue Router 路由管理
- Vite 构建工具
- JavaScript (非TypeScript)

## 主要功能
- ✅ 响应式设计，完美适配桌面端和移动端
- ✅ 精美的智能体卡片展示
- ✅ 实时搜索和分类筛选
- ✅ 白色/暗黑主题切换
- ✅ 点击卡片跳转到Dify智能体页面
- ✅ 智能体数据存储在JSON文件中，通过Pinia管理

## 项目结构
```
src/
├── components/          # 组件目录
│   ├── AppHeader.vue   # 应用头部组件
│   ├── AgentCard.vue   # 智能体卡片组件
│   └── SearchFilter.vue # 搜索筛选组件
├── stores/             # Pinia状态管理
│   ├── agents.js       # 智能体数据管理
│   └── theme.js        # 主题管理
├── data/               # 数据文件
│   └── agents.json     # 智能体数据
├── views/              # 页面组件
│   └── HomeView.vue    # 首页
├── assets/             # 静态资源
│   └── main.css        # 全局样式
├── router/             # 路由配置
│   └── index.js
└── main.js             # 应用入口
```

## 开发命令
- `npm run dev` - 启动开发服务器
- `npm run build` - 构建生产版本
- `npm run preview` - 预览生产版本

## 部署说明
1. 运行 `npm run build` 构建项目
2. 将 `dist` 目录部署到Web服务器
3. 确保服务器支持SPA路由（配置404重定向到index.html）

## 数据管理
- 智能体数据存储在 `src/data/agents.json` 文件中
- 通过Pinia store (`src/stores/agents.js`) 进行状态管理
- 支持搜索、分类筛选等功能

## 主题系统
- 支持浅色和深色主题自动切换
- 主题状态保存到localStorage
- 遵循系统主题偏好设置
- 使用CSS变量实现主题切换

## 响应式设计
- 桌面端：多列网格布局
- 平板端：适中的卡片排列
- 移动端：单列垂直布局
- 针对不同屏幕尺寸优化交互体验

---
> Source: [chen-banxia/dify-application-sample](https://github.com/chen-banxia/dify-application-sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
