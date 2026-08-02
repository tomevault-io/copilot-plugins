## fast-soy-admin

> FastSoyAdmin v1.0.0 | FastAPI + Vue3 全栈后台管理模板 | MIT

# FastSoyAdmin - Claude Code Guide

FastSoyAdmin v1.0.0 | FastAPI + Vue3 全栈后台管理模板 | MIT

后端 [app/](app/)（FastAPI/Python），前端 [web/](web/)（Vue3/TypeScript，pnpm workspace），部署 [deploy/](deploy/)，迁移 [migrations/](migrations/)。

文档（**所有约定细节看这里，本文不再重复**）：

- 在线：<https://sleep1223.github.io/fast-soy-admin-docs/>
- 离线：[docs/](docs/) — 与在线一致的 Markdown 镜像
- 独立文档站源码：[.extra_repo/fast-soy-admin-docs/src/](.extra_repo/fast-soy-admin-docs/src/) — 更新公开文档时与 `docs/` 同步
- 项目说明：[README.md](README.md) / [docs/](docs/)

---

## 重要文件（动手前先看）

- **[.env](.env)** — 运行配置事实来源（`SECRET_KEY` / `DB_URL` / `REDIS_URL` / `JWT_*`）。从 [.env.example](.env.example) 复制；**不要**提交。
- **[justfile](justfile)** — 所有常用命令入口。`just --list` 列全部；不要绕过它直调底层命令。
- **[app/core/](app/core/)** — 框架级代码，影响整个后端。改这里前读 [init_app.py](app/core/init_app.py)、[base_schema.py](app/core/base_schema.py)、[crud.py](app/core/crud.py)、[router.py](app/core/router.py)。
- **[app/utils/**init**.py](app/utils/**init**.py)** — 业务模块统一 import 入口；新增 core 能力要给 business 用必须在这里 re-export。
- **[app/system/services/init_helper.py](app/system/services/init_helper.py)** — `ensure_menu` / `ensure_role` / `reconcile_menu_subtree` / `refresh_api_list`。**业务模块允许从这里 import**，是少数显式暴露的 system service。
- **`DB_URL` 指向的数据库** — 默认依赖含 `tortoise-orm[asyncpg]` + `aiosqlite`，PostgreSQL/SQLite 开箱即用；切 MySQL/MSSQL/Oracle 需 `uv sync --extra {mysql|mssql|oracle}`。**不要**直接 SQL 改它绕过模型/迁移；走 `just mm`。

**Python 工具链** — 后端依赖与脚本一律走 `uv`：`uv sync` / `uv add <pkg>` / `uv run <cmd>`。**不要**直接调 `python` / `pip` / `python -m xxx`，**不要**用系统/pyenv 的 `python`。

> **Trust by verify**：真实状态有时与配置/代码漂移（手工迁移、未跑 `mm`、Redis 缓存未刷新）。动手前用 `just dbhistory`、读 `.env`、看 Redis key、跑 `just check` 核对一遍。

---

## 标准操作流程

### 任何修改之前

1. **明确范围**：system 还是 business？哪个模块？影响哪些表 / 路由 / 角色？
2. **看清现状**：`git status`、`just dbhistory`、读相关 `models.py` / `init_data.py`
3. **跟规范对齐**：[工程约定清单](#工程约定清单pr-review-checklist) + [docs/standard/](docs/standard/)
4. **小步推进**：先生成迁移看 SQL 再 apply；高风险 schema / contract 变更补覆盖；用户明确禁止测试时只跑允许的检查并说明跳过项

### 提交 / 推送之前

提交或推送 Git 前必须先跑项目级门禁：默认执行 `just check`，若门禁自动修复或暴露问题，先处理并重新跑到通过后再提交 / 推送。用户明确禁止测试时，只跑允许的格式化 / 静态检查并在结果中说明跳过项。

### 常用命令

```bash
just install                          # 装后端 (uv sync) + 前端 (pnpm install)
cp .env.example .env                  # 首次复制环境变量
just db-init                          # 首次初始化数据库

just run                              # 并行启动后端 :9999 + 前端 :9527
just run backend / just run frontend  # 仅后端 / 仅前端

just mm                               # makemigrations + migrate（启动不会自动迁移）
just fmt                              # 格式化并应用安全 lint 修复（前后端统一入口）
just check                            # 提交前门禁（前后端统一入口）
just up / just logs / just down       # docker compose
```

完整命令清单见 [docs/reference/commands.md](docs/reference/commands.md)。

### 新增业务模块

```bash
just cli-init <name>              # 生成骨架
# 编辑 app/business/<name>/models.py 定义模型
just cli-crud crm 客户管理 "--yes --models Customer --data-scope Customer:owner_id,tenant_id --button-auth"
just mm && just run && just fmt && just check
```

业务模块结构、autodiscover 约定见 [docs/develop/autodiscover.md](docs/develop/autodiscover.md)。

### 启动初始化与对账

每次启动由 Redis leader worker 串行执行：`init_menus()` → `refresh_api_list()` → `init_users()` → 各业务模块 `init_data.init()` → `refresh_all_cache()`。

⚠️ **IaC 模式陷阱**：启用了 `reconcile_menu_subtree(...)` 的子树是单一数据源，通过 Web UI 在该子树下手工创建的菜单/按钮**会在下次重启时被清除**。允许用户动态创建菜单的子树**不要**调用它。

⚠️ **漂移告警**：`ensure_role` 引用的 `route_name` / `button_code` / `(method, path)` 解析失败会 `log.warning`——必须立即修复，否则角色权限静默缺失。

完整说明见 [docs/develop/init-data.md](docs/develop/init-data.md)。

### 事故响应

1. **先读日志**：`docker compose logs -f`、Radar 面板（`/manage/radar/*`）、[app/core/exceptions.py](app/core/exceptions.py) 业务码
2. **看 Redis 状态**：leader worker key、缓存 key、限流 key
3. **未确认前不动数据**：尤其是 truncate / drop / 反向迁移 / 直接改数据库文件
4. **建议恢复方案再执行**：先汇报、给选项、等用户拍板

---

## 权限边界

### 始终允许（无需确认）

- 读取仓库文件、`.env.example`、`justfile`、`pyproject.toml`、`web/package.json`
- 跑只读 / `--check` / dry-run 命令
- `just --list` / `just dbhistory` / `just fmt` / `just check`
- 查看监控、日志、Radar 面板
- 读取系统表 / 元数据：`information_schema.*`、Tortoise meta

### 需要用户确认（先问再做）

- **框架级**：编辑 `app/core/*`、`app/utils/__init__.py` 的对外 re-export、`init_app.py` 全局中间件/异常处理器/路由挂载、`app/core/code.py` 已有响应码（追加新码不需要确认）、`justfile` 已有 target 语义、`pyproject.toml` / `web/package.json` 主版本依赖升级
- **数据库**：`migrate`（先给用户看 SQL）、手写迁移文件、改 `BaseModel` / `AuditMixin` / `SoftDeleteMixin` / `TreeMixin`
- **RBAC / 安全**：改 `init_data.py` 菜单/按钮/角色种子、`R_SUPER` 行为或 `DependPermission` 校验逻辑、`app/core/data_scope.py` 行级规则、`JWT_*` / `GUARD_*` / `CORS_ORIGINS`
- **部署 / 服务**：改 `deploy/`、`docker-compose.yml`、`nginx.conf`，重启服务、`just up` / `just down`

### 禁止 — 始终需要明确授权

破坏性操作必须用户明确说"确认/yes"：

```
- DROP DATABASE / DROP TABLE / TRUNCATE / DELETE 不带 WHERE
- 反向迁移 / 手工 downgrade
- rm -rf migrations/ / rm app_system.sqlite3
- git reset --hard / git push --force（非个人 feature 分支）
- 删除业务模块目录 app/business/<name>/
- 直接 SQL 改用户/角色/权限表绕过 init_data
- 清空 Redis（FLUSHDB / FLUSHALL）
```

**业务数据访问限制**：默认**不读业务表内容**；排查问题前问授权；即便授权也用 `LIMIT` + 指定列；不要把业务数据写到日志 / 临时文件。

**确认流程**：操作的具体表 / 模块 / 文件？有没有近期备份？让用户输入完整名字确认（如 `truncate biz_hr_employee` 让用户键入 `biz_hr_employee`）。

---

## 架构速览

两个顶层包：[app/system/](app/system/)（认证/RBAC/用户/角色/菜单/API/字典/监控）+ [app/business/<name>/](app/business/)（业务，启动时 [autodiscover.py](app/core/autodiscover.py) 自动发现）。每个模块统一分层：`api/` → `services/` → `controllers/` → `models + schemas`。

**依赖方向**（铁律）：

- `app/core/` 不依赖 system / business
- `app/system/` 仅依赖 `app/core/`
- `app/business/<x>/` 依赖 `app/utils`，**不得**反向 import `app.system.*`（白名单：`init_helper` 显式暴露的 service），**不得**互相 import 兄弟业务模块——跨模块走事件总线（`emit` / `on`）

**前端**：Vue3 + Vite + Naive UI + Elegant Router + Pinia + UnoCSS + Alova。

完整架构、分层职责、关键 core 文件清单见 [docs/getting-started/architecture.md](docs/getting-started/architecture.md) 与 [docs/develop/intro.md](docs/develop/intro.md)。

---

## 工程约定清单（PR review checklist）

> 各项展开见 [docs/develop/](docs/develop/) 与 [docs/standard/](docs/standard/)。

这些约定分三层：**边界不可破坏**，默认实现优先复用，复杂业务允许按扩展点显式改写。

### 不可破坏边界

1. 响应统一用 `Success` / `SuccessExtra` / `Fail`；不要返回裸 dict，不手拼前端字段命名。
2. 业务 schema 继承 `SchemaBase`；分页继承 `PageQueryBase`；对外 ID 用 `SqidId` / `SqidPath`；Update schema 用 `make_optional`。
3. 业务模块 import 入口统一 `from app.utils import ...`；不得直接 import 兄弟业务模块；跨模块联动走事件总线（`emit` / `on`）。
4. `app/core/` 不依赖 system / business；`app/system/` 不反向依赖 business；业务模块仅允许使用 system 显式暴露的 service（如 `init_helper`）。
5. 写接口必须有后端权限校验（按钮权限或等价依赖）；不要靠"前端隐藏按钮"做安全；不要在业务里硬判 `role_code == "..."`，用 `has_role_code` / `has_button_code`。
6. 业务角色种子显式声明 `data_scope`，公开值只有 `all / scope / self / custom`。
7. 行级权限使用 `scope_id` + `scope_id_field` 表达业务范围；具体字段可映射为 `tenant_id`、`project_id`、`store_id` 等。
8. 事务用 `in_transaction(get_db_conn(Model))`，不要硬编码连接名；事务内不要做 HTTP / Redis / 队列等外部 IO。
9. 不要 `raise HTTPException`；业务失败用 `BizError` / `SchemaValidationError`。
10. 关键节点、权限拒绝、重要安全事件写 `radar_log(...)`；高频调试用 `log.debug`；不要 `print(...)`。

### 默认实现策略

1. 简单资源优先使用 `CRUDRouter` + `CRUDBase` 生成标准 6 路由；自定义标准动作用 `@crud.override`，不要绕过 `_OrderedRouter`。
2. 聚合根或有副作用的资源（订单、工单、审批、状态机等）用显式 `@router.*` + `services/`；当 override ≥ 3 或出现事务、Redis、跨模型写、事件、审计时，及时下沉到 service。
3. `controllers` / `services` 保持框架无关，不 import `fastapi.Request` / `Response`；api 层只做参数接收、依赖注入、响应封装。
4. 模型默认继承 `BaseModel + AuditMixin`；字段写 `description`；`Meta.table = biz_<module>_<entity>`；需要软删/树结构时显式加 `SoftDeleteMixin` / `TreeMixin`。
5. `ForeignKeyField` / `OneToOneField` 上方声明 `<name>_id: int`（或 `int | None`）注解；创建、更新、比较优先用 `obj.<name>_id`；访问关系对象字段前先 `prefetch_related(...)` 或 `await obj.<name>`。
6. 业务自有缓存按 `<module>_<resource>:<scope>` 命名，读 → miss → 查 → 写 TTL，变更时主动失效；分页列表不要直接套全局 `@cache(...)`，除非 key 已包含用户/范围/查询参数。
7. 所有函数加类型注解；默认交付前跑 `just fmt` + `just check`。用户明确禁止测试时，不跑 `just check`，改跑允许的格式化/静态检查并说明。

### 可扩展点

1. CLI 生成的 `_get_scope_id()` 是业务接入钩子，生成后按模块上下文替换实现。
2. `--data-scope Model:user_id,scope_id` 的第二字段只是默认名，可指定 `tenant_id`、`project_id`、`store_id` 等任意模型字段。
3. `--list-cache` 与 `--rate-limit` 可按模型开启；列表缓存必须确认不会跨用户或跨 scope 复用。
4. 新功能示例优先用 `crm` / `inventory` / `ticket` 等中性模块；main 分支不保留 HR 示例代码或文档。
5. 输出文件路径统一使用 `/`，避免在文档、日志和生成结果中混用 Windows 反斜杠。

---
> Source: [sleep1223/fast-soy-admin](https://github.com/sleep1223/fast-soy-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
