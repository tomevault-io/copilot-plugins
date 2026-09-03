## memoryball

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

MemoryTree是一个革命性的3D交互式照片画廊，结合了沉浸式WebGL图形和前沿的手势识别技术。该项目将照片浏览体验转变为直观、神奇的3D球体记忆之旅。

## 核心技术栈

- **Three.js** (v0.181.2): 3D图形渲染和场景管理
- **TypeScript**: 类型安全的现代JavaScript开发
- **MediaPipe Tasks Vision** (@mediapipe/tasks-vision): Google的手势识别和追踪
- **Vite** (v7.2.4): 快速开发服务器和构建工具
- **lil-gui**: 轻量级UI控制面板

## 项目结构

```
memorytree/
├── src/
│   ├── core/                   # 核心3D系统
│   │   ├── ThreeScene.ts      # Three.js场景管理
│   │   ├── PhotoSystem.ts     # 照片布局和交互系统
│   │   └── ExplosionSystem.ts # 视觉效果系统
│   ├── vision/                # 手势追踪
│   │   └── VisionManager.ts   # MediaPipe集成
│   ├── ui/                    # 用户界面
│   │   └── UIManager.ts       # UI控件和反馈
│   ├── shapes/                # 3D几何
│   │   └── ShapeGenerator.ts  # 照片定位生成
│   └── main.ts               # 应用入口点
├── public/
│   └── photos/               # 照片集合 (130+ 张图片)
├── index.html                # 主HTML文件
├── package.json             # 依赖和脚本
├── tsconfig.json            # TypeScript配置
└── MANUAL.md                # 中文操作手册
```

## 开发命令

### 基础开发
```bash
# 安装依赖
npm install

# 启动开发服务器 (热重载)
npm run dev

# 生产构建
npm run build

# 预览生产构建
npm run preview
```

### 开发服务器访问
- 开发模式: http://localhost:5173
- 支持热重载和快速开发迭代

## 系统架构组件

### ThreeScene类 (`src/core/ThreeScene.ts`)
- WebGL渲染器设置和管理
- 摄像机控制和透视
- 动画循环和性能优化
- 窗口调整和响应式行为
- 自适应像素比设置(最大2倍)

### PhotoSystem类 (`src/core/PhotoSystem.ts`)
- 照片纹理加载和缓存
- 3D球体布局定位
- 状态管理(CRYSTAL_BALL → SPHERE → DETAIL)
- 平滑过渡和动画
- 射线投射交互检测
- 照片悬停和选择效果

### VisionManager类 (`src/vision/VisionManager.ts`)
- MediaPipe手部关键点检测
- 手势识别算法(PINCH, SWIPE, FIST等)
- 实时光标追踪
- 手部坐标标准化
- 左右手分离控制逻辑
- 8秒超时处理和降级模式

### UIManager类 (`src/ui/UIManager.ts`)
- 屏幕光标渲染
- 控制面板界面
- 状态信息显示
- 用户反馈系统

## 交互状态系统

应用包含三个主要状态：

1. **CRYSTAL_BALL**: 初始水晶球状态
   - 照片凝聚成球体
   - 任何捏合手势触发爆炸到球体状态

2. **SPHERE**: 球体探索模式
   - 照片分布在3D球面上
   - 左手控制缩放(捏合距离)
   - 右手控制旋转和选择
   - 双击捏合进入详情模式

3. **DETAIL**: 单图详情模式
   - 全屏查看单张照片
   - 左右滑动切换照片
   - 握拳返回球体模式

## 手势控制系统

### 双手缩放控制 (NEW)
- **合掌缩放**: 双手同时控制球体缩放
  - 双手合掌：球体缩小到最小尺寸
  - 双手分开：球体放大到最大尺寸
- **距离映射**: 3D空间中的双手距离映射到缩放因子
- **平滑处理**: 使用30%平滑因子避免抖动
- **检测要求**: 需要双手同时被MediaPipe检测

### 右手控制
- **移动**: 控制球体3D旋转
- **捏合**: 选择照片/进入详情模式
- **双捏合**: 快速进入详情模式
- **左滑/右滑**: 在详情模式下切换照片
- **握拳**: 在详情模式下返回球体

## 开发注意事项

### TypeScript配置
- 启用严格类型检查
- 使用现代ES6+特性
- 模块化架构设计

### 性能优化
- 高效的纹理管理
- 60FPS渲染目标
- 内存使用优化
- 射线投射检测优化

### 错误处理
- 优雅降级(摄像头不可用时使用鼠标控制)
- MediaPipe加载超时处理(8秒)
- 网络连接问题的演示模式
- 初始化失败的友好提示

### 照片资源
- 130+张高质量图片集合
- 存储在`public/photos/`目录
- 自动纹理加载和缓存
- 响应式缩放适配不同设备

## 浏览器兼容性

- **Chrome**: 完全支持，最佳性能
- **Firefox**: 良好支持，性能略有差异
- **Safari**: 支持，可能需要权限提示
- **Edge**: 近期版本支持

## 故障排除要点

- 确保8秒内加载MediaPipe模型
- 摄像头权限检查和错误处理
- WebGL硬件加速要求
- 网络连接问题的演示模式切换
- 光照条件对手势识别的影响

## 开发最佳实践

- 保持模块化架构和关注点分离
- 优先使用现有工具和库
- 遵循项目的TypeScript严格模式
- 确保每个提交都能编译和通过测试
- 优先考虑用户体验和性能优化

---
> Source: [VAAN0524/memoryball](https://github.com/VAAN0524/memoryball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
