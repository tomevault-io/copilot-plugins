## prd-reader

> 这是一个基于 Electron 的 PRD（产品需求文档）阅读器应用，主要功能包括：

# Electron PRD 阅读器项目规则

## 项目概述
这是一个基于 Electron 的 PRD（产品需求文档）阅读器应用，主要功能包括：
- 导入本地 PRD 文件夹
- 启动内置 HTTP 服务器
- 使用 Chromium 内核浏览器渲染 HTML 文件
- 提供文件浏览和实时预览功能

## 项目结构

### 核心文件
- [package.json](mdc:package.json) - 项目依赖和脚本配置
- [main.js](mdc:main.js) - Electron 主进程入口文件
- [preload.js](mdc:preload.js) - 预加载脚本，处理主进程与渲染进程通信
- [index.html](mdc:index.html) - 主窗口 HTML 文件
- [renderer.js](mdc:renderer.js) - 渲染进程脚本
- [styles.css](mdc:styles.css) - 应用样式文件

### 服务器相关
- [server/http-server.js](mdc:server/http-server.js) - HTTP 服务器实现
- [server/file-watcher.js](mdc:server/file-watcher.js) - 文件监听器（可选）

### 工具类
- [utils/file-utils.js](mdc:utils/file-utils.js) - 文件操作工具函数
- [utils/path-utils.js](mdc:utils/path-utils.js) - 路径处理工具函数

## 技术栈要求

### 核心依赖
- **Electron**: 主框架，版本建议 >=20.0.0
- **express**: HTTP 服务器框架
- **serve-static**: 静态文件服务中间件
- **chokidar**: 文件监听（可选，用于热重载）

### 开发依赖
- **electron-builder**: 用于打包应用
- **nodemon**: 开发时自动重启（可选）

## 开发规范

### 文件命名
- 使用 kebab-case 命名文件和文件夹
- JavaScript 文件使用 `.js` 扩展名
- 配置文件使用描述性名称

### 代码规范
- 使用 ES6+ 语法
- 异步操作优先使用 async/await
- 错误处理必须包含 try-catch 块
- 所有用户交互都要有加载状态和错误提示

### 安全考虑
- 在 [preload.js](mdc:preload.js) 中暴露安全的 API
- 禁用 nodeIntegration，启用 contextIsolation
- 验证用户输入的文件路径和端口号

## 功能实现指导

### 主进程 ([main.js](mdc:main.js))
- 创建主窗口
- 处理应用生命周期
- 管理 HTTP 服务器启动/停止
- 处理文件夹选择对话框

### 渲染进程 ([renderer.js](mdc:renderer.js))
- 文件夹选择界面
- 文件列表展示（左侧面板）
- 浏览器视图控制（右侧面板）
- 端口配置界面

### HTTP 服务器 ([server/http-server.js](mdc:server/http-server.js))
- 使用 Express 创建静态文件服务器
- 支持自定义端口
- 处理 CORS 问题
- 提供服务器状态监控

### 文件操作 ([utils/file-utils.js](mdc:utils/file-utils.js))
- 递归读取文件夹结构
- 过滤 HTML 文件
- 验证文件路径有效性
- 生成文件树结构

## UI/UX 设计原则

### 布局结构
- 左侧：文件浏览器面板（30% 宽度）
- 右侧：Chromium 浏览器面板（70% 宽度）
- 顶部：工具栏（文件夹选择、端口配置、服务器状态）

### 交互设计
- 点击 HTML 文件自动在右侧浏览器中加载
- 提供刷新按钮重新加载当前页面
- 显示当前服务器地址和端口
- 支持拖拽文件夹到应用窗口

### 样式要求 ([styles.css](mdc:styles.css))
- 使用现代扁平化设计
- 深色主题优先
- 响应式布局
- 清晰的视觉层次

## 错误处理

### 常见错误场景
- 端口被占用
- 文件夹路径无效
- 文件读取权限不足
- HTTP 服务器启动失败

### 错误处理策略
- 所有错误都要有用户友好的提示
- 提供错误恢复建议
- 记录详细错误日志
- 优雅降级处理

## 性能优化

### 文件加载
- 大文件夹使用虚拟滚动
- 图片资源懒加载
- 缓存文件列表结构

### 内存管理
- 及时清理不用的 BrowserView
- 限制同时打开的文件数量
- 监控内存使用情况

## 打包和分发

### 构建配置
- 使用 electron-builder 进行打包
- 支持 Windows、macOS、Linux
- 生成安装包和便携版本

### 资源优化
- 压缩静态资源
- 排除开发依赖
- 最小化应用体积

## 开发流程

### 启动开发环境
```bash
npm install
npm run dev
```

### 构建应用
```bash
npm run build
npm run dist
```

### 调试指南
- 使用 Chrome DevTools 调试渲染进程
- 使用 console.log 调试主进程
- 启用 Electron 开发者工具

## 注意事项

1. **路径处理**: 确保跨平台路径兼容性
2. **端口管理**: 检查端口可用性，提供端口冲突解决方案
3. **文件监听**: 可选实现文件变化自动刷新功能
4. **安全性**: 验证用户输入，防止路径遍历攻击
5. **用户体验**: 提供清晰的状态反馈和操作指导

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cnzhihao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
