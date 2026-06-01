## interview-helper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

AI面试助手是一个基于React的网页应用，帮助用户进行面试准备。系统使用科大讯飞RTasr API提供实时语音转文字功能，并通过DeepSeek API提供AI驱动的回答建议。项目采用前后端分离架构。

## 开发命令

### 前端命令（在 `/frontend` 目录下执行）
```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 运行代码检查
npm run lint

# 预览生产构建
npm run preview
```

### 根目录命令
```bash
# 安装根目录依赖
npm install
```

### 后端设置和启动
后端使用FastAPI框架，需要PostgreSQL和Redis数据库支持。

#### 环境要求
- Python 3.9+
- PostgreSQL 17+ (Docker容器)
- Redis 7+ (Docker容器)

#### 安装和启动步骤
```bash
# 进入后端目录
cd backend

# 创建虚拟环境
python3 -m venv venv

# 激活虚拟环境
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt

# 配置环境变量（.env文件已包含默认配置）
# DATABASE_URL=postgresql://postgres:root@localhost:5432/ai_interview_helper
# REDIS_URL=redis://localhost:6379

# 初始化数据库
python test_db.py  # 创建数据库和测试连接
python init_default_user.py  # 创建默认用户(test/test1234)

# 启动服务器
python run_server.py  # 运行在 http://localhost:9000
```

#### 默认用户账户
- 用户名：`test`
- 密码：`test1234`
- 邮箱：`test@example.com`

#### API端点
- 健康检查：`GET /health`
- 用户认证：`POST /api/auth/login`, `POST /api/auth/register`
- 会话管理：`GET/POST /api/sessions/`
- API文档：`http://localhost:9000/docs`

## 架构设计

### 前端技术栈
- **框架**: React 19 + TypeScript + Vite
- **UI组件**: Ant Design (antd)
- **状态管理**: Zustand配合Immer中间件和DevTools
- **路由**: React Router DOM v7
- **样式**: Styled Components + CSS
- **HTTP客户端**: Axios配合自定义请求拦截器和JWT认证

### 后端技术栈
- **框架**: FastAPI + Uvicorn
- **数据库**: PostgreSQL 17 + SQLAlchemy ORM
- **缓存**: Redis 7
- **认证**: JWT (JSON Web Tokens)
- **API文档**: 自动生成Swagger/OpenAPI文档
- **数据验证**: Pydantic模型

### 关键目录结构

#### 前端目录结构
- `src/api/` - API集成层（DeepSeek、科大讯飞RTasr、后端API）
- `src/components/` - 可复用UI组件
- `src/contexts/` - React上下文
- `src/layouts/` - 布局组件（MainLayout）
- `src/pages/` - 路由级页面组件
- `src/store/` - Zustand状态存储（interviewStore、authStore、apiConfigStore、userConfigStore）
- `src/types/` - TypeScript类型定义
- `src/utils/` - 工具函数

#### 后端目录结构
- `app/` - FastAPI应用主目录
  - `models.py` - SQLAlchemy数据模型
  - `schemas.py` - Pydantic请求/响应模型
  - `auth.py` - JWT认证和密码处理
  - `database.py` - 数据库连接配置
  - `routers/` - API路由模块
    - `auth.py` - 用户认证路由
    - `sessions.py` - 会话管理路由
- `main.py` - FastAPI应用入口
- `requirements.txt` - Python依赖
- `.env` - 环境变量配置

### 状态管理架构
应用使用四个主要的Zustand存储：
- `interviewStore.ts` - 管理面试会话、消息和转写结果，包含会话持久化功能
- `authStore.ts` - 处理用户认证状态、登录/注册/登出
- `apiConfigStore.ts` - 处理不同提供商的API配置
- `userConfigStore.ts` - 用户偏好和设置

### 核心功能

#### 智能问题检测系统
- **轻量级预检查**：避免过度触发，只在接近阈值时进行完整检查
- **主动静音检查**：音频变为静音1.1秒后自动触发AI问题检测
- **被动数据检查**：语音转录数据中断超过3秒时触发检测
- **防抖机制**：500ms防抖确保不重复触发
- **KV Cache优化**：充分利用DeepSeek的上下文缓存，降低70-90%成本

#### 语音转文字集成
- 使用科大讯飞RTasr WebSocket API进行实时音频转写
- 通过RecorderManager处理音频流
- 处理并显示具有角色分离的转写结果
- 智能语音活动检测，追踪静音持续时间

#### AI集成与优化
- DeepSeek API集成，支持历史上下文和KV Cache
- 滑动窗口策略（30秒+30条消息）优化性能
- 消息标记系统避免重复处理（isAsked属性）
- 稳定前缀设计最大化缓存命中率
- 容错处理：语音识别错误容忍、角色混淆处理、内容碎片拼接

#### 路由结构
- `/interview/meeting` - 主要面试会话页面（全屏优化布局）
- `/interview/new` - 面试设置和配置
- `/settings` - API配置和系统设置
- `/` - 首页（占位符）

## 技术实现要点

### 智能问题检测架构
- **预检查过滤**：`shouldPreCheck()` 轻量级时间窗口过滤（>2.5s）
- **双路径触发**：被动检查（语音转录时）+ 主动检查（静音延迟）
- **绕过机制**：主动检查通过 `bypassPreCheck=true` 绕过预检查
- **全局状态管理**：`silenceStartTime`、`lastDataReceiveTime` 追踪时间状态
- **资源清理**：组件卸载时自动清理定时器和回调

### 音频处理与状态同步
- 通过 `/public/xfyunRtasr/` worker文件集成WebAudio API
- `setAudioActiveState()` 同步音频状态到问题检测系统
- `updateAudioActiveState()` 追踪静音持续时间
- 实时音频流处理和WebSocket通信

### API集成与缓存优化
- **KV Cache策略**：稳定前缀 + 增量追加设计
- **消息准备**：`prepareMessagesForAI()` 分离历史和新消息
- **持久化存储**：xfyunConfigStore 和 apiConfigStore 支持 localStorage
- **错误容错**：动态导入避免循环依赖，完善的错误处理

### UI架构与布局
- **固定底部操作栏**：`flexShrink: 0` 确保不被压缩
- **分离滚动区域**：聊天区和语音区独立滚动，`minHeight: 0` 启用滚动
- **自动滚动**：`useEffect` 监听数据变化自动滚到底部
- **响应式设计**：Flexbox布局适配不同屏幕尺寸

### 构建配置
- Vite配合React插件
- 启用TypeScript严格模式
- 配置路径别名（`@` 指向 `src/`）
- ESLint配合React特定规则

## 外部依赖

### 核心API
- **科大讯飞RTasr**: 实时语音识别WebSocket API
- **DeepSeek**: AI聊天完成API用于生成响应

### 关键库
- `antd` - UI组件库
- `zustand` - 轻量级状态管理
- `axios` - HTTP客户端
- `react-router-dom` - 路由
- `styled-components` - CSS-in-JS样式
- `react-markdown` - Markdown渲染
- `crypto-js` - API认证加密函数

## 开发工作流

### 智能问题检测工作流
1. **语音转录处理**：科大讯飞RTasr实时转写语音为文本
2. **智能触发检测**：
   - 被动检查：语音转录数据接近3秒中断时触发预检查
   - 主动检查：音频静音1.1秒后自动触发检测
3. **KV Cache优化调用**：准备消息上下文，利用DeepSeek缓存机制
4. **AI响应生成**：基于滑动窗口历史对话生成回答
5. **实时显示**：通过markdown渲染进行流式显示
6. **状态更新**：标记已处理消息，优化后续缓存命中

## 记忆库上下文

项目包含 `/memory-bank` 目录，记录了：
- 项目路线图和版本规划
- 技术上下文和架构决策
- 实现进度和任务跟踪
- 样式指南和开发模式

这是一个处于v0.3阶段的活跃开发项目，已完成后端架构和数据持久化功能，下一步将专注于多模型支持和用户系统完善。

## v0.3 更新说明

### 主要新增功能
- **完整的后端架构**: 基于FastAPI + PostgreSQL + Redis
- **用户认证系统**: JWT认证，支持注册/登录/登出
- **会话持久化**: 所有面试会话和消息自动保存到数据库
- **默认测试账户**: test/test1234，便于快速测试
- **前后端集成**: 完整的API客户端和状态管理
- **智能问题检测系统**: 基于AI语义理解的高精度问题识别
- **KV Cache优化**: DeepSeek API调用成本降低70-90%
- **API配置持久化**: 支持XFYUN和DeepSeek配置本地存储
- **设置界面**: 完整的API配置和系统设置页面
- **UI布局优化**: 固定底部操作栏，分离滚动区域，全屏适配

### 数据库模型
- **Users表**: 用户基本信息和认证数据
- **InterviewSessions表**: 面试会话元数据
- **SessionMessages表**: 会话中的所有消息记录

### API接口
- **认证接口**: `/api/auth/login`, `/api/auth/register`, `/api/auth/me`
- **会话接口**: `/api/sessions/` (CRUD操作)  
- **消息接口**: `/api/sessions/{id}/messages` (消息管理)

### 安全特性
- **敏感信息保护**: 所有API密钥和tokens通过环境变量或应用内设置管理
- **版本控制安全**: .env文件和敏感配置已加入.gitignore
- **JWT安全**: 使用强密钥进行token签名
- **密码加密**: 使用bcrypt加密存储用户密码

### v0.3.1 智能问题检测系统

#### 核心特性
- **语义理解检测**: 摒弃规则匹配，使用AI语义分析识别问题
- **触发时机优化**: 双路径触发机制（被动+主动）避免漏检和过度检测
- **成本控制**: KV Cache充分利用，API调用成本降低70-90%
- **容错处理**: 语音识别错误容忍，角色混淆智能处理

#### 技术实现
- **轻量级预检查**: `shouldPreCheck()` 只在接近阈值时触发完整检查
- **静音检测**: 音频静音1.1秒后主动触发AI问题检测
- **防抖机制**: 500ms防抖避免重复触发
- **滑动窗口**: 30秒+30条消息优化上下文长度
- **消息标记**: `isAsked` 属性避免重复处理
- **资源清理**: 组件卸载时自动清理定时器和回调

#### 架构优化
- **异步动态导入**: 避免循环依赖，使用 `await import()` 延迟加载
- **浏览器兼容**: `window.setTimeout` 替换 `NodeJS.Timeout` 类型
- **状态同步**: 音频活跃状态与问题检测系统实时同步
- **错误处理**: 完善的异常捕获和用户友好的错误提示

#### 文件结构
- `src/utils/questionDetection.ts` - 问题检测核心逻辑和工具函数
- `src/api/xfyunRtasr.ts` - 语音转录与智能触发集成
- `src/api/deepseek.ts` - KV Cache优化的DeepSeek API调用
- `src/store/interviewStore.ts` - 消息状态管理和持久化
- `src/pages/InterviewMeeting.tsx` - UI布局优化和状态同步

### v0.3.2 设置系统与配置管理

#### 持久化存储
- **xfyunConfigStore**: 科大讯飞API配置，支持localStorage持久化
- **apiConfigStore**: DeepSeek API配置，支持localStorage持久化
- **配置验证**: 实时状态检查和错误提示
- **默认值**: 智能默认配置减少用户配置负担

#### 设置界面
- **标签页设计**: XFYUN和DeepSeek配置分离，界面清晰
- **表单验证**: Ant Design Form组件，实时验证和错误提示
- **状态显示**: 配置状态实时反馈，连接成功/失败提示
- **导航集成**: 主菜单添加设置入口，便于快速访问

### v0.3.3 UI布局优化

#### 全屏适配
- **100vh高度**: 完整利用视口高度，消除默认边距
- **固定底部操作栏**: `flexShrink: 0` 确保操作栏不被压缩
- **分离滚动区域**: 聊天区和语音区独立滚动，`minHeight: 0` 启用滚动
- **自动滚动**: 消息和答案更新时自动滚到底部

#### 布局结构
- **Flexbox架构**: 主内容区 `flex: 1`，底部操作栏固定高度
- **响应式设计**: 聊天区和语音区比例优化，适配不同屏幕
- **视觉层次**: 问答分栏显示，用户问题左侧，AI回答右侧
- **状态指示**: 录音状态、音频活跃状态实时显示

#### 交互优化
- **音频状态同步**: 录音状态与音频活跃状态实时同步显示
- **操作反馈**: 按钮状态变化、加载状态、错误提示完善
- **键盘快捷键**: 支持快捷操作，提升用户体验

---
> Source: [JasonJarvan/interview-helper](https://github.com/JasonJarvan/interview-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
