## bosszp-outsourcing-marker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个**Boss直聘外包公司标记项目**，包含Chrome浏览器插件（前端）和外包公司管理API（后端）。项目采用前后端分离架构，帮助用户在Boss直聘网站自动识别并标记外包公司。

- **前端**: Chrome Extension (Manifest V3)
- **后端**: Vercel Serverless Functions
- **数据库**: Redis
- **AI服务**: DeepSeek V3
- **技术栈**: Node.js 18+, ES modules, JavaScript/HTML/CSS

## 常用开发命令

### 后端服务（主要开发环境）

```bash
# 进入后端目录
cd server

# 安装依赖
npm install

# 启动本地开发服务器
npm run dev

# 部署到Vercel生产环境
npm run deploy

# 初始化Redis数据（从 outsourcing-companies.json 导入）
npm run init-redis

# 重置Redis数据
npm run reset-redis

# 测试AI判断功能
npm run test-ai

# 初始化Edge Config
npm run init-edge-config

# 重置Edge Config
npm run reset-edge-config
```

### Chrome插件开发

插件使用原生JavaScript开发，无需构建步骤：
1. 直接在Chrome中加载根目录（开启开发者模式）
2. 修改文件后刷新插件即可生效
3. 使用`popup.html`/`popup.js`进行用户交互
4. 使用`content_script.js`注入到Boss直聘页面

## 核心架构

### 前端架构（Chrome插件）

**入口文件**:
- `manifest.json`: 插件配置（权限、注入范围、内容安全策略）
- `content_script.js`: 页面注入核心脚本，负责DOM监听和公司标记
- `popup.html` / `popup.js`: 用户交互界面（切换开关、刷新数据）
- `styles.css`: 视觉样式

**关键流程**:
```
页面加载 → 获取公司列表(优先缓存) → DOM匹配 → 添加红色标签
           ↓
        MutationObserver监听动态内容 → 实时标记
```

**缓存机制**:
- 存储位置: `chrome.storage.local`
- 缓存键: `outsourcingCompaniesCache`, `outsourcingCompaniesCacheExpiry`
- 有效期: 24小时
- 手动刷新: 用户可主动更新缓存

### 后端架构（Serverless API）

**目录结构**:
```
server/
├── api/                    # 路由层
│   ├── companies.js       # GET /api/companies - 获取公司列表
│   ├── add-company.js     # POST /api/add-company - 添加公司
│   └── health.js          # 健康检查
├── services/              # 业务逻辑层
│   └── ai-service.js      # AI服务封装（DeepSeek V3）
├── config/                # 配置层
│   ├── ai-config.js       # AI服务配置
│   └── prompt-templates.js # 提示词模板
├── scripts/               # 工具脚本
│   └── init-redis-companies.js  # Redis初始化
└── index.js               # 根路由（重定向到/api）
```

**数据流**:
```
客户端请求 → Vercel Serverless Function → Redis查询/存储 → 返回响应
    ↓
POST /api/add-company → AI判断(DeepSeek V3) → 验证通过 → 存储到Redis
```

**API端点**:

| 方法 | 路径 | 功能 |
|------|------|------|
| GET | `/api/companies` | 获取外包公司列表 |
| POST | `/api/add-company` | 添加新公司（需AI验证） |
| GET | `/api/health` | 健康检查 |

**API响应格式**:

```json
// GET /api/companies
{
  "outsourcing_companies": ["公司1", "公司2"],
  "count": 2,
  "updated_at": "2025-11-17T..."
}

// POST /api/add-company
{
  "message": "公司添加成功",
  "company": "公司名称",
  "total": 100
}
```

## 环境配置

### 必需环境变量（server/.env.local）

```env
REDIS_URL=redis://username:password@host:port
AI_BASE_URL=https://api.deepseek.com/v1
AI_MODEL=deepseek-v3
AI_TEMPERATURE=0.2
HY_TOKEN=your_api_token
HY_USER=your_user_id
HY_SOURCE=web
AGENT_ID=your_agent_id
```

**注意**: `.env.local`已被Git忽略，不会提交到版本库。配置时参考`server/.env.local`示例。

## 开发工作流

### 1. 本地开发

```bash
# 启动后端服务
cd server && npm run dev

# 测试API
curl http://localhost:3000/api/companies

# 测试AI功能
cd server && npm run test-ai
```

### 2. 数据库管理

```bash
# 首次部署前初始化Redis
cd server && npm run init-redis

# 从静态文件重新导入数据
npm run reset-redis
```

数据来源: `server/outsourcing-companies.json`

### 3. Chrome插件测试

1. 打开 `chrome://extensions/`
2. 开启"开发者模式"
3. 点击"加载已解压的扩展程序"
4. 选择项目根目录
5. 访问zhipin.com测试功能

### 4. 生产部署

```bash
cd server
# 配置环境变量到Vercel Dashboard
npm run deploy
npm run init-redis  # 部署后初始化数据
```

## 关键文件说明

### 核心业务文件

- **`content_script.js`**: 页面注入核心逻辑
  - `fetchCompanies()`: 获取公司列表
  - `markOutsourcingCompanies()`: DOM标记
  - `MutationObserver`: 监听动态内容

- **`services/ai-service.js`**: AI服务封装
  - DeepSeek V3 API调用
  - 流式/非流式响应支持
  - 自动联网搜索公司信息

- **`api/add-company.js`**: 添加公司API
  - 参数验证
  - AI判断调用
  - Redis存储

- **`config/prompt-templates.js`**: AI提示词模板
  - 外包公司判断逻辑
  - 提示词优化

### 配置与数据

- **`manifest.json`**: Chrome插件配置
  - 注入域名: `https://zhipin.com/*`
  - 权限: storage, scripting, tabs
  - CSP策略: 限制外部资源

- **`vercel.json`**: Vercel部署配置
  - CORS设置
  - 资源限制

- **`server/outsourcing-companies.json`**: 静态公司列表（Redis数据源）

## 重要特性

### 1. AI智能判断

- 使用DeepSeek V3模型联网搜索
- 自动判断是否为外包公司
- 防止恶意标记非外包公司
- 温度参数: 0.2（保证准确性）

### 2. 缓存机制

- 前端: chrome.storage.local（24小时）
- 后端: Redis高性能缓存
- 手动刷新: 支持用户主动更新

### 3. 动态适配

- MutationObserver监听页面变化
- 支持无限滚动加载
- 实时标记新内容

## 当前项目状态

- **版本**: v1.2.0（详见RELEASE.md）
- **最近更新**: 支持缓存外包名单，必要时手动刷新
- **技术优化**: AI提示词优化、AI服务异常处理

## 注意事项

1. **Boss直聘页面结构变化**: 可能需要更新content_script.js中的DOM选择器
2. **AI准确性**: 提交公司时请使用完整名称
3. **API限制**: Vercel可能有请求限制
4. **网络连接**: 插件需访问互联网获取公司列表
5. **隐私**: 插件仅在Boss直聘运行，不收集个人信息

## 代码风格要求

1. **前端组件**: 遵循原子设计原则，组件需原子化拆分
2. **命名规范**: 驼峰命名法(camelCase)或PascalCase
3. **注释**: 代码需包含丰富的中文注释
4. **模块化**: 清晰的文件/文件夹结构组织

## 技术债务与待优化项

- 可考虑添加单元测试（当前无测试框架）
- 可增加API身份验证机制
- 可优化DOM选择器稳定性
- 可添加错误监控和日志系统

---
> Source: [evepupil/BOSSzp-outsourcing-marker](https://github.com/evepupil/BOSSzp-outsourcing-marker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
