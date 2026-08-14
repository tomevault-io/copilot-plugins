## pachong-cursor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个全栈商品调研工具，使用 FastAPI + Vue3 + Playwright 构建，能够自动爬取淘宝商品数据并生成调研报告。

## 开发命令

### 后端开发
```bash
# 环境准备（首次运行）
python -m venv venv
source venv/bin/activate  # Linux/Mac
pip install -r requirements.txt
playwright install chromium
cp env.example .env

# 启动开发服务器
python -m uvicorn backend.app.main:app --reload --port 8000

# 运行测试
python -m pytest backend/tests/

# 调试工具
python debug_crawler.py
python test_extractor.py
python quick_verification.py
```

### 前端开发
```bash
# 环境准备（首次运行）
cd frontend
npm install
cp env.example .env

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览构建结果
npm run preview

# 运行测试
npm run test          # 运行一次性测试
npm run test:watch    # 监听模式测试
```

## 核心架构

### 后端架构（FastAPI）
- **`backend/app/main.py`**: 应用入口点，路由定义和中间件配置
- **`backend/services/`**: 业务逻辑层
  - `task_service.py`: 任务管理核心服务
  - `crawler_orchestrator.py`: 爬虫协调器，管理爬取流程
  - `data_extractor.py`: 数据提取器，从页面提取商品信息
  - `browser_service.py`: Playwright 浏览器管理
  - `mock_data_service.py`: 模拟数据服务（开发/演示模式）
- **`backend/models/`**: 数据模型定义
- **`backend/repositories/`**: 数据访问层
- **`backend/schemas/`**: API 请求/响应模式
- **`backend/config.py`**: 配置管理（基于 Pydantic Settings）

### 前端架构（Vue 3）
- **`frontend/src/main.ts`**: 应用入口
- **`frontend/src/App.vue`**: 根组件
- **`frontend/src/views/`**: 页面组件
  - `SearchView.vue`: 搜索页面
  - `ResultView.vue`: 结果展示页面
- **`frontend/src/api/`**: API 调用封装

### 核心工作流程
1. **任务创建**: 用户输入关键词，系统创建异步爬取任务
2. **爬取协调**: `crawler_orchestrator` 管理整个爬取流程
3. **浏览器自动化**: 使用 Playwright 控制浏览器访问淘宝
4. **数据提取**: `data_extractor` 从页面提取商品信息
5. **智能回退**: 网络问题时自动切换到模拟数据模式
6. **报告生成**: 生成 Markdown 格式的调研报告

## 技术特点

### 异步任务处理
- 使用 FastAPI 的异步特性处理长时间运行的爬取任务
- 前端通过轮询获取任务状态和进度

### 智能数据回退
- 配置 `ENABLE_MOCK_DATA=true` 启用模拟数据模式
- 生产环境遇到反爬虫时自动回退到模拟数据

### 配置管理
- 后端使用 Pydantic Settings 管理配置
- 支持环境变量和 `.env` 文件
- 关键配置项：浏览器设置、爬取延时、安全限制

### 安全措施
- 路径遍历保护（文件下载）
- 文件类型限制
- 关键词长度限制
- CORS 配置

## 测试和调试

### 可用的调试脚本
- `quick_verification.py`: 快速验证系统功能
- `debug_crawler.py`: 调试爬取流程
- `test_extractor.py`: 测试数据提取器

### 测试覆盖
- 后端：使用 pytest，包含 API、服务层和数据层测试
- 前端：使用 Vitest，包含组件和 API 测试

## 开发注意事项

### 模拟数据模式
开发时建议启用模拟数据模式避免网络问题：
```bash
# 在 .env 中设置
ENABLE_MOCK_DATA=true
```

### 数据库
- 默认使用 SQLite (`data.sqlite3`)
- 生产环境建议切换到 PostgreSQL

### 浏览器配置
- 开发模式可设置 `BROWSER_HEADLESS=false` 查看浏览器操作
- 调整 `CRAWL_DELAY_MIN/MAX` 控制爬取速度

## API 访问
- 后端 API: http://localhost:8000
- API 文档: http://localhost:8000/docs
- 前端应用: http://localhost:5173

---
> Source: [liaolj/pachong-cursor](https://github.com/liaolj/pachong-cursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
