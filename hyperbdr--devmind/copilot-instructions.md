## devmind

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

DevMind 是一个 AI 驱动的企业内部门户平台，包含 Django REST API 后端（cloud billing、data collector、AI price hub）和配套的 Vue 3 前端。核心开发在 `backend/`（Django）和 `frontend/`（Vue）两个目录。

## 常用命令

### 初始化（克隆后必执行）
```bash
git submodule update --init --recursive   # 拉取 agentcore 子模块
```

### Docker 本地开发
```bash
cp env.sample .env.dev
docker-compose -f docker-compose.dev.yml up -d
# Web: http://localhost:8000
# API docs: http://localhost:8000/swagger/
# Celery monitor (Flower): http://localhost:5555
```

### Docker 生产构建
```bash
cp env.sample .env
docker-compose up -d
# 默认端口: HTTP 10080, HTTPS 10443
```

### Python 开发（非 Docker）
```bash
pip install -e .[dev]
# 本地 agentcore editable 模式:
for d in backend/agentcore/*/; do [ -f "${d}pyproject.toml" ] && pip install -e "$d"; done
```

### 测试与代码质量
```bash
pytest                              # 运行所有测试
pytest path/to/test.py              # 单个测试文件
python backend/manage.py test        # Django test runner（部分测试）
black --check backend/              # 检查格式
isort --check backend/              # 检查 import 顺序
```

### Django 管理命令
```bash
python backend/manage.py migrate
python backend/manage.py register_periodic_tasks   # 注册所有定时任务
python backend/manage.py createsuperuser
```

### 前端
```bash
cd frontend
npm install
npm run dev          # 开发服务器
npm run build        # 生产构建
npm run lint         # ESLint
npm run test:e2e     # Playwright E2E
```

## 架构概览

### Django 应用结构（backend/）

每个 Django app 都是自包含的：拥有自己的 models、views、serializers、services、migrations、periodic_tasks 和 tests。跨 app 的回归测试放在 `backend/tests/`。

主要 App：
- **`accounts/`** — 用户认证、JWT、OAuth（Google）、Profile、Role、Permission
- **`cloud_billing/`** — 云账单管理（AWS/Azure/阿里云/华为云/腾讯云），providers 目录包含各云厂商适配器
- **`data_collector/`** — 从 JIRA、飞书等系统拉取原始数据，支持 attachments 和统计 API
- **`ai_pricehub/`** — AI 模型价格对比，LangChain + LLM parser，支持 APScheduler 同步
- **`app_config/`** — 全局配置（Feature Flag）

共享基础设施（`backend/core/`）：
- **`settings/`** — Django 配置分片（base.py、database.py、celery.py、rest.py、swagger.py、accounts.py、cache.py、ai_services.py 等）
- **`urls.py`** — 根路由，mount 所有 app 的 URL
- **`celery.py`** — Celery app 配置，含 autodiscover_tasks() 自动发现各 app 的 tasks.py
- **`periodic_registry.py`** — 定时任务注册器，所有 app 的 periodic_tasks 通过 `register_periodic_tasks()` 写入 django_celery_beat

### agentcore 子模块（git submodules under `backend/agentcore/`）

agentcore 是独立维护的包，通过 git submodule 引入。子模块被当作 Django app 使用：

| 子模块 | INSTALLED_APPS | 路由前缀 | 功能 |
|---|---|---|---|
| `agentcore-metering` | `agentcore_metering.adapters.django` | `/api/v1/admin/` | LLM 用量追踪 |
| `agentcore-task` | `agentcore_task.adapters.django` | `/api/v1/tasks/` | 统一任务管理 |
| `agentcore-notifier` | `agentcore_notifier.adapters.django` | `/api/v1/admin/notifications/` | 飞书通知 |

### 定时任务机制（Celery Beat）

- **Task 发现**：`core/celery.py` 调用 `app.autodiscover_tasks()`，自动加载所有 INSTALLED_APPS 中各 app 的 `tasks.py`
- **Periodic Task 注册**：不通过代码声明，而是在启动时由 `register_periodic_tasks` management command 发现各 app 的 `periodic_tasks.register_periodic_tasks()` 函数，写入 django_celery_beat 数据库。**现有记录不会被覆盖**，以保留运维人员在数据库中的自定义修改
- **容器入口**：`docker/entrypoint.sh` 在 gunicorn 容器中运行 `register_periodic_tasks` 一次；celery-beat 容器使用 DatabaseScheduler 从数据库读取调度

### API 层

- **REST Framework**：DRF + drf-spectacular（OpenAPI schema、Swagger UI、ReDoc）
- **认证**：dj-rest-auth (JWT) + django-allauth (OAuth Google) + allauth headless API（前后端分离场景）
- **国际化**：Django i18n，中文（zh-hans）和英文（en），通过 `LanguageCodeMappingMiddleware` 映射浏览器 Accept-Language
- **时区**：所有数据存储为 UTC，前端负责用户本地时区转换

### 前端（frontend/）

- Vue 3 + Vite + Composition API + `<script setup>`
- Pinia（状态管理）、Vue Router、vue-i18n、Tailwind CSS
- E2E：Playwright（`playwright.config.cjs`）
- 前端构建集成在根 `Dockerfile` 的 `frontend` target 中

## 编码规范（来自 Cursor rules）

- **响应语言**：所有解释用中文，代码注释用英文
- **注释规则**：无行内注释，注释写在代码块上方；类和函数使用 docstring（triple quotes）
- **行宽**：每行最多 79 字符
- **Import 结构**：三段式（stdlib → third-party → local app），段内按字母排序，不混用
- **业务逻辑位置**：放在 models、serializers、services 中，views 只处理请求
- **Django/DRF**：优先使用 CBV（复杂逻辑）和 DRF 内置功能，不手写原始 SQL
- **调试用 print**：避免使用，用 logging 代替

## 安全与配置

- 不提交 `.env`、secrets、证书；从 `env.sample` 复制创建
- 生产使用 PostgreSQL（推荐）、开发可用 SQLite
- `docker/nginx/certs/` 和云厂商配置发布前需审查

---
> Source: [HyperBDR/devmind](https://github.com/HyperBDR/devmind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
