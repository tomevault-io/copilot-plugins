## zurl

> Zurl 是一个短链接管理系统，使用以下技术栈：

# AGENTS.md - Zurl 开发指南

## 项目概述

Zurl 是一个短链接管理系统，使用以下技术栈：
- **后端**: Python 3 + FastAPI + SQLAlchemy + SQLite + Redis
- **前端**: Vue 3 + Element Plus + Pinia + Vite
- **部署**: Docker

## 项目结构

```
zurl/
├── app/                    # 后端代码
│   ├── api/               # API 业务逻辑层
│   ├── config.py          # 配置管理
│   ├── main.py            # FastAPI 应用入口
│   ├── middleware/        # 中间件（认证、点击统计等）
│   ├── models/            # SQLAlchemy 数据模型
│   ├── routers/           # 路由定义
│   └── utils/             # 工具函数
├── frontend/              # 前端代码
│   ├── src/
│   │   ├── components/    # Vue 组件
│   │   ├── stores/        # Pinia 状态管理
│   │   ├── utils/         # 工具函数
│   │   └── views/         # 页面视图
│   └── vite.config.js
└── docker-compose.yaml
```

## 构建和运行命令

### 后端

```bash
# 安装依赖
pip install -r app/requirements.txt

# 启动开发服务器
uvicorn app.main:app --reload --host 0.0.0.0 --port 3080

# 数据库迁移
alembic upgrade head
```

### 前端

```bash
cd frontend

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev

# 构建生产版本
pnpm build
```

### Docker

```bash
# 构建并启动
docker-compose up -d

# 查看日志
docker-compose logs -f
```

### 测试

当前项目**没有**自动化测试。如需添加测试：

```bash
# 后端测试（如添加 pytest）
pip install pytest pytest-asyncio
pytest tests/

# 前端测试（如添加 vitest）
pnpm add -D vitest
pnpm test
```

## 代码风格指南

### Python 后端

#### 导入顺序
1. 标准库导入
2. 第三方库导入
3. 本地应用导入

```python
import time
import re
from datetime import datetime

from fastapi import APIRouter, Form, Request, Depends
from pydantic import BaseModel
from sqlalchemy import desc

from app.models.conn import get_db
from app.utils.helper import show_json, md5
```

#### 命名约定
- **函数/变量**: `snake_case`（如 `get_client_ip`, `short_url`）
- **类名**: `PascalCase`（如 `UrlAPI`, `UserItem`）
- **常量**: `UPPER_SNAKE_CASE`（如 `DENY_SHORT_URLS`, `DB_FILE_PATH`）
- **数据库表名**: `zurl_` 前缀（如 `zurl_urls`, `zurl_sessions`）

#### API 响应格式
使用 `show_json` 统一返回格式：
```python
from app.utils.helper import show_json

return show_json(200, "success", data)
return show_json(400, "error.message", {})
return show_json(404, "not.found", {})
```

#### 错误处理
- 使用 HTTPException 抛出认证错误
- 业务逻辑错误通过 `show_json` 返回错误码
- 数据库操作使用 try/finally 确保连接关闭

```python
db = next(get_db())
try:
    # 数据库操作
    result = db.query(Model).filter(...).first()
    return show_json(200, "success", result)
finally:
    db.close()
```

#### Pydantic 模型
定义在对应 API 文件顶部：
```python
from pydantic import BaseModel, HttpUrl, EmailStr

class UrlItem(BaseModel):
    short_url: str = None
    long_url: str
    title: str = None
    ttl_days: int = 0
```

### Vue 前端

#### 组件结构
使用 `<script setup>` 语法糖，按以下顺序组织：
1. `<template>`
2. `<script setup>`
3. `<style scoped>`

#### 导入顺序
1. Vue 相关（ref, onMounted 等）
2. 第三方库（Element Plus, axios 等）
3. 本地组件/stores/utils

```vue
<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { useI18n } from 'vue-i18n'

import req from '@/utils/req'
import { useSiteStore } from '@/stores/site'
import MyComponent from '@/components/MyComponent.vue'
</script>
```

#### 命名约定
- **组件文件**: `PascalCase.vue`（如 `AdminView.vue`, `list.vue`）
- **变量/函数**: `camelCase`（如 `shortUrl`, `getPosts`）
- **Pinia stores**: `camelCase`（如 `site.js`, `base.js`）
- **常量**: `UPPER_CASE`

#### 状态管理
使用 Pinia，定义在 `stores/` 目录：
```javascript
import { defineStore } from "pinia";

export const useSiteStore = defineStore('site', {
    state: () => ({
        // 状态
    }),
    actions: {
        // 方法
    }
})
```

#### HTTP 请求
使用封装的 axios 实例：
```javascript
import req, { toForm } from '@/utils/req'

// GET 请求
req.get("/api/urls?page=1&limit=10")

// POST JSON
req.post("/api/search", { filter: "short_url", keyword: "test" })

// POST FormData
req.post("/api/delete/url", toForm({ short_url: "test" }))
```

#### 国际化
使用 vue-i18n，文案使用 `$t()` 或 `t()`：
```vue
<template>
    <span>{{ $t('link.list') }}</span>
</template>

<script setup>
const { t } = useI18n()
ElMessage.success(t('success'))
</script>
```

## 数据库约定

- 使用 SQLAlchemy ORM
- 时间戳存储为 Unix 时间戳（整数）
- 数据库文件路径：`app/data/db/zurl.db`
- 配置文件路径：`app/data/config.toml`

## 常见任务

### 添加新的 API 端点
1. 在 `app/api/` 创建/修改 API 类
2. 定义 Pydantic 请求模型（如需要）
3. 在 `app/routers/routers.py` 添加路由
4. 使用 `get_current_session` 依赖保护需要认证的接口

### 添加新的前端页面
1. 在 `src/views/` 创建视图组件
2. 在 `src/router/index.js` 添加路由配置
3. 在管理后台则修改 `src/components/admin/` 组件

## 注意事项

- 后端代码使用中文注释是可接受的
- 前端需要支持中英双语（使用 vue-i18n）
- 敏感配置不要提交到仓库
- 修改配置后需要重启服务

---
> Source: [helloxz/zurl](https://github.com/helloxz/zurl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
