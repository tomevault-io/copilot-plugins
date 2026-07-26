## app2docker

> app2docker 项目核心架构与构建流程上下文


⚠️ 本文件由 AI 自动生成，项目结构变更后请重新执行生成命令

## 项目概述

App2Docker：可视化 Docker 镜像构建平台。FastAPI 后端 + Vue3 前端，单容器部署（挂载 `data/` + `docker.sock`）。支持文件上传/Git 源码构建、流水线、镜像导出、SSH/Agent/Portainer 部署。

当前版本：`backend/VERSION` → **1.0.2**；前端 `frontend/package.json` → **1.0.2**

---

## 【技术栈】

### 后端（Python 3）
| 依赖 | 版本 | 用途 |
|------|------|------|
| fastapi | 0.115.6 | HTTP API |
| uvicorn[standard] | 0.34.0 | ASGI 服务 |
| sqlalchemy | 2.0.23 | ORM（SQLite） |
| docker | 7.1.0 | Docker API / buildx |
| PyJWT | 2.10.1 | JWT 认证 |
| PyYAML | 6.0.3 | 配置/部署 YAML |
| paramiko | 3.4.0 | SSH 部署 |
| httpx | 0.27.0 | Webhook 触发 |
| websockets | 14.1 | Agent WebSocket 客户端 |
| psutil | 6.1.0 | Agent 主机信息采集 |
| croniter | 5.0.1 | 流水线定时调度 |
| watchdog | 5.0.3 | 文件监听 |

入口：`backend/app.py`（`python backend/app.py` 或 Dockerfile `CMD`）

### 前端（Node 20 + Vue 3）
| 依赖 | 版本 | 用途 |
|------|------|------|
| vue | ^3.5.24 | UI 框架 |
| vite | ^5.4.0 | 构建/开发服务器（port 3000，proxy `/api` → 8000） |
| axios | ^1.13.2 | HTTP 客户端 |
| bootstrap | ^5.3.8 | UI 组件 |
| tailwindcss | ^4.2.2 | 样式 |
| jwt-decode | ^4.0.0 | 前端 JWT 解析 |
| codemirror / vue-codemirror | ^6.x | YAML/Dockerfile 编辑器 |

构建产物：`frontend/vite.config.js:37-40` → `outDir: '../dist'`

### 基础设施
- 数据库：SQLite `data/app2docker.db`（WAL 模式，`backend/database.py:12-14`）
- 运行时数据：`data/`（config.yml、uploads、docker_build、exports、templates、logs、secret_key.txt）
- 内置模板：`templates/{jar,nodejs,python,go,web}/`
- Docker 镜像：`Dockerfile` 多阶段（frontend-builder → backend-base → app2docker / app2docker-agent）

---

## 【目录结构】

> 本项目无根级 `src/`；前端源码在 `frontend/src/`，后端在 `backend/`。

```
frontend/src/                    # Vue 前端（3层）
├── App.vue                      # 根组件、菜单路由
├── main.js                      # 入口，挂载 axios 拦截器
├── components/                  # 功能面板（按菜单划分）
│   ├── StepBuildPanel.vue       # 镜像构建
│   ├── PipelinePanel.vue        # 流水线
│   ├── DataSourcePanel.vue      # Git 数据源
│   ├── TaskManager.vue          # 任务管理
│   ├── TemplatePanel.vue        # 模板管理
│   ├── DeployTaskManager.vue    # 部署管理
│   ├── AgentHostManager.vue     # Agent 主机
│   ├── HostManager.vue          # SSH 主机
│   ├── PortainerDeployManager.vue
│   ├── LoginPage.vue / UserManagement.vue / RoleManagement.vue
│   └── ...                      # Dashboard、Export、Registry、Docker 等
├── composables/
│   └── useModalEscape.js
└── utils/
    ├── auth.js                  # Token 存取
    ├── axios-interceptor.js     # 401 跳转登录
    ├── permissions.js           # 前端权限过滤菜单
    └── projectTypes.js          # 项目类型（与 backend/project_types.py 同步）

backend/                         # Python 后端（非 src/）
├── app.py                       # FastAPI 入口、startup 初始化
├── routes.py                    # 全部 REST + WebSocket 路由（/api 前缀）
├── handlers.py                  # 构建/导出/模板核心业务
├── docker_builder.py            # Local/Remote/Mock Docker 构建器
├── config.py                    # data/config.yml 读写
├── auth.py                      # JWT + 用户/权限
├── database.py / models.py      # SQLite ORM
├── git_source_manager.py        # Git 数据源 CRUD
├── pipeline_manager.py          # 流水线
├── scheduler.py                 # cron 调度
├── template_parser.py           # {{VAR}} 模板变量替换
├── project_types.py             # jar/nodejs/python/go/web
├── websocket_handler.py         # Agent WS 连接管理
├── deploy_executors/            # agent / ssh / portainer 部署执行器
│   └── factory.py
└── agent/                       # 远程 Agent 进程
    ├── main.py
    └── websocket_client.py

# 省略：node_modules/、dist/、.git/、data/（运行时）、docs/、test/、scripts/
templates/                       # 内置 Dockerfile 模板（只读）
├── jar/                         # dragonwell8/17/21 build+upload
├── nodejs/                      # nodejs18/20
├── python/                      # python39/310/311/312
├── go/                          # go1.21/22/23
└── web/                         # nginx-simple
```

---

## 【Docker 镜像构建流程】

### A. App2Docker 自身镜像（平台容器）

```
Dockerfile
├── 阶段1 frontend-builder (L5-35)
│   COPY frontend/ → npm install → npm run build → /app/dist
├── 阶段2 backend-base (L39-113)
│   Alpine + Python venv + requirements.txt + COPY backend/
├── 阶段3 app2docker-agent (L116-148, --target app2docker-agent)
│   CMD backend/agent/start.sh
└── 阶段4 app2docker 默认 (L151-194)
    COPY --from=frontend-builder /app/dist → ./dist
    COPY templates/ → ./templates/
    ENV APP_PORT=8000
    CMD python backend/app.py
```

独立 Agent 镜像：`agent.Dockerfile`（等价于主 Dockerfile 的 agent 阶段）

### B. 用户应用镜像构建（核心业务链）

#### 路径1：文件上传构建
```
POST /api/upload                          routes.py:2019
  → BuildManager.start_build()            handlers.py:1410
  → 后台线程 _build_task()                handlers.py:1464
      ├── 写入 data/uploads + 解压到 data/docker_build/{image}_{taskId}
      ├── get_template_path()             handlers.py:177 → templates/ 或 data/templates/
      ├── parse_template() 替换 {{VAR}}   template_parser.py:112 + handlers.py:2840
      ├── create_docker_builder()         docker_builder.py:1102
      │     ├── use_remote=true → RemoteDockerBuilder (L622)
      │     ├── 否则 LocalDockerBuilder (L463) → docker buildx build
      │     └── 均不可用 → MockDockerBuilder (L1050)
      └── docker_builder.build_image()    handlers.py:1937
            → _build_with_buildx()        docker_builder.py:178-460
            → 可选 push_image()           handlers.py:2166
```

#### 路径2：Git 源码构建
```
POST /api/build-from-source               routes.py:2509
  → BuildTaskManager 创建 task_type=build_from_source
  → git clone（source_id 时用 GitSourceManager 认证）
  → 同上：模板渲染 → docker_builder.build_image()
  → 支持多服务（parse_dockerfile_services）、resource_package 注入
```

#### 路径3：流水线触发
```
Pipeline (DB) → webhook/cron/manual
  → pipeline_manager.py + scheduler.py
  → pipeline_to_task_config()             handlers.py:4465
  → BuildTaskManager 入队 → 同上构建链
```

#### Docker 构建器初始化
```
handlers.py 模块加载                      handlers.py:54-67
  init_docker_builder()
  → load_config()                         config.py:65
  → create_docker_builder(docker_config)  docker_builder.py:1102
```

#### 关键运行时目录（handlers.py:34-41）
| 路径 | 用途 |
|------|------|
| `data/uploads/` | 上传文件 |
| `data/docker_build/` | 构建上下文（临时） |
| `data/exports/` | 导出 tar |
| `templates/` | 内置模板（只读） |
| `data/templates/` | 用户自定义模板 |

---

## 【数据源配置】

### 支持的类型
| 类型 | 存储 | 管理类 | API 前缀 |
|------|------|--------|----------|
| **Git 仓库** | SQLite `git_sources` 表 | `GitSourceManager` | `/api/git-sources` |
| **全局 Git 凭据** | `data/config.yml` → `git:` 段 | `config.get_git_config()` | `/api/config` |
| **SSH 主机** | SQLite `hosts` 表 | `HostManager` | `/api/hosts` |
| **Agent 主机** | SQLite `agent_hosts` 表 | `AgentHostManager` | `/api/agent-hosts` |
| **Portainer** | `agent_hosts`（host_type=portainer） | `PortainerExecutor` | 同上 |
| **镜像仓库** | `data/config.yml` → `docker.registries[]` | `config.py` | `/api/registries` |

### Git 数据源字段（models.py:163-182）
`source_id, name, git_url, branches, tags, default_branch, username, password(加密), dockerfiles(JSON)`

### 加载机制
1. **Git 数据源**：`GitSourceManager`（`git_source_manager.py`）→ CRUD 走 SQLAlchemy；密码 AES 加密（`crypto_utils.encrypt_password`，密钥派生自 `SECRET_KEY`）
2. **全局配置**：`config.py:9` → `CONFIG_FILE = "data/config.yml"`；启动时 `ensure_config_exists()`（app.py:224）；`load_config()` 自动合并默认值并兼容旧单仓库格式
3. **构建时认证**：`build-from-source` 传 `source_id` 时调用 `GitSourceManager.get_auth_config()`（git_source_manager.py:279）
4. **部署配置**：`deploy_configs` 表 + YAML 示例 `deploy-config.yaml`；解析 `deploy_config_parser.py`

### 项目类型（构建模板分类）
`jar | nodejs | python | go | web` — 定义于 `backend/project_types.py:7-43`，前端镜像 `frontend/src/utils/projectTypes.js`

---

## 【鉴权模式】

### 认证方式（三层）
1. **JWT Bearer Token**（主）：`Authorization: Bearer <token>`；登录 `POST /api/login`（routes.py:259）→ `auth.authenticate()`（auth.py:97）→ `create_token()` HS256，有效期 24h（auth.py:13）
2. **APP Key**：`X-API-Key` 头或 Bearer 内联；`app_key_manager.validate_app_key()`；`verify_token()` 在 JWT 无效时回退（auth.py:86-91）
3. **Agent WebSocket**：`/api/ws/agent?token=` 或 `?secret_key=&agent_token=`（routes.py:8940）；密钥表 `agent_secrets`；待批准主机走 `pending_host_manager`

### 密钥/凭据存储
| 密钥 | 路径/位置 |
|------|-----------|
| JWT 签名密钥 | `data/secret_key.txt`（auth.py:16,19-47） |
| Registry/Git/SSH/Portainer 密码 | DB 或 config.yml，AES-GCM 加密（crypto_utils.py:10-20，密钥=SHA256(SECRET_KEY)） |
| Agent token | `agent_hosts.token`（64 字符） |
| 默认管理员 | DB `users` 表，初始密码 `admin`（SHA256 哈希） |

### 中间件/挂载点
| 层级 | 位置 | 行为 |
|------|------|------|
| CORS | `backend/app.py:26-32` | 允许所有来源 |
| API 路由前缀 | `backend/app.py:35` | `app.include_router(router, prefix="/api")` |
| 路由级鉴权 | `backend/routes.py:146` | `require_auth(request)` — 检查 Bearer / X-API-Key |
| 权限装饰器 | `backend/auth.py:268-335` | `@require_permission("menu.xxx")` / `@require_role` |
| 公开端点 | routes.py | `/api/public/version`、`/api/login`、webhook 端点（token 校验非 JWT） |
| 前端拦截 | `frontend/src/utils/axios-interceptor.js` | 401 → 清 token 跳登录 |
| 前端 Token | `frontend/src/utils/auth.js` | localStorage |

### RBAC
- 表：`users` → `user_roles` → `roles` → `role_permissions` → `permissions`
- 权限码：`backend/permissions.py`（如 `menu.build`、`menu.datasource`）
- 默认角色：admin（全权限）、user（除 menu.users）、readonly（只读菜单）

---

## 【常见开发任务索引】

### 新增 Git 数据源字段/逻辑
| 改动点 | 文件 |
|--------|------|
| DB 模型 | `backend/models.py` → `GitSource` |
| DB 迁移 | `backend/database.py` → 新增 migrate 函数 |
| CRUD | `backend/git_source_manager.py` |
| API | `backend/routes.py` → `/git-sources` 段（~6854） |
| UI | `frontend/src/components/DataSourcePanel.vue` |

### 修改镜像构建逻辑
| 改动点 | 文件 |
|--------|------|
| 上传构建入口 | `backend/routes.py:2019` + `backend/handlers.py:1410`（BuildManager） |
| Git 构建入口 | `backend/routes.py:2509` + `backend/handlers.py:4691`（BuildTaskManager） |
| 构建上下文/模板渲染 | `backend/handlers.py:1464`（_build_task）、`:2840`（parse_template） |
| Docker 引擎抽象 | `backend/docker_builder.py` |
| 模板变量语法 | `backend/template_parser.py` |
| 内置/用户模板 | `templates/`、`data/templates/` |
| 新增项目类型 | `backend/project_types.py` + `frontend/src/utils/projectTypes.js` + 新模板目录 |
| 流水线触发 | `backend/pipeline_manager.py`、`backend/scheduler.py` |
| 构建 UI | `frontend/src/components/StepBuildPanel.vue`、`BuildConfigEditor.vue` |

### 调整鉴权/权限
| 改动点 | 文件 |
|--------|------|
| JWT/登录/密码 | `backend/auth.py` |
| APP Key | `backend/app_key_manager.py` |
| 路由鉴权 helper | `backend/routes.py:146`（require_auth） |
| 权限码定义 | `backend/permissions.py` |
| 权限检查/装饰器 | `backend/auth.py:193-335` |
| 用户/角色 API | `backend/routes.py` → users/roles 段 |
| DB 初始化角色权限 | `backend/database.py:migrate_add_user_system` |
| 前端登录/Token | `frontend/src/utils/auth.js`、`LoginPage.vue` |
| 前端菜单过滤 | `frontend/src/utils/permissions.js`、`App.vue` |

### 新增部署目标类型
| 改动点 | 文件 |
|--------|------|
| 执行器基类 | `backend/deploy_executors/base.py` |
| 工厂注册 | `backend/deploy_executors/factory.py:27` |
| Agent 端执行 | `backend/agent/deploy_executor.py` |
| 部署 YAML 解析 | `backend/deploy_config_parser.py` |
| 示例配置 | `deploy-config.yaml` |

### 本地开发
```bash
./dev.sh              # 后端 8000 + 前端 3000
./init.sh             # 初始化 data/ 目录
docker build -t app2docker .   # 生产镜像
```

---
> Source: [numen06/app2docker](https://github.com/numen06/app2docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
