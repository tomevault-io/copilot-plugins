## overseas-ai-sales

> > 目标：让任意新 AI 在接手时，能稳定产出符合本项目习惯的改动。

# 乔门智脑项目 AI 协作规则

> 目标：让任意新 AI 在接手时，能稳定产出符合本项目习惯的改动。

## 1. 基本原则

- 先理解链路，再改代码：优先确认“消息从哪里来、在哪里处理、最终发到哪里”。
- 小步改动、可回滚：每次变更应尽量聚焦一个问题，避免大范围无关重构。
- 不破坏现有线上流程：涉及 iPad 协议、自动回复、鉴权、工作流执行时，优先兼容。
- 出现异常先加可观测性：必要时先补日志，再做逻辑修复。

## 2. 代码风格与结构习惯

- 后端：
  - 使用 FastAPI + SQLAlchemy async 既有风格。
  - 业务逻辑放 `services/`，路由层只做参数校验与组装响应。
  - 新增配置统一放 `app/config.py`，禁止硬编码密钥。
  - 日志统一 `logging.getLogger("qiaomen.*")`，日志应包含关键上下文（ID/状态/路径）。
- 前端：
  - React + TypeScript + Ant Design 风格优先。
  - API 调用统一走 `src/api/request.ts` 的 axios 实例。
  - 全局状态优先放 Zustand（`stores/`），避免深层 props 传递。
  - 新页面使用 `App.tsx` 懒加载路由，菜单同步更新 `MainLayout.tsx`。

## 3. 必须遵守的业务规则

- iPad 协议自动回复链路：
  - `wecom_ipad_service.poll_and_save()` 是入站消息核心入口。
  - 仅处理有效文本消息并做好去重，防止重复回复。
  - 自动回复必须有并发保护（按联系人锁）和短时间去重窗口。
- AI 回复过滤：
  - 只拦截“AI 自曝身份”类措辞，不可误杀正常销售话术。
  - 修改 `BANNED_PHRASES` 时必须评估业务影响并记录验证结果。
- 工作流执行：
  - 兼容旧版节点结构，新增字段应保持向后兼容。
  - 任何工作流节点行为变更都要给出最小回归用例。

## 4. 安全与配置规则

- 严禁提交以下内容到仓库：
  - `.env`、生产密钥、服务器密码、API Token、私钥文件。
- 所有敏感信息必须来自环境变量或密钥管理系统。
- 若发现历史明文敏感信息：
  - 第一时间提示风险
  - 建议迁移到环境变量
  - 建议轮换密钥

## 5. 变更流程规范

- 修改前：
  - 先阅读 `README.md`、`ARCHITECTURE.md`、`HANDOVER.md`。
  - 明确影响范围（前端/后端/部署/数据）。
- 修改中：
  - 只改与任务相关文件，避免无关格式化和重命名。
  - 保持 API 向后兼容（尤其 `/api/wecom/*`、`/api/ipad-callback/*`）。
- 修改后：
  - 至少执行关键链路验证（健康检查、登录、消息收发、AI 回复）。
  - 在 `HANDOVER.md` 补充“根因-修改-验证-风险”。

## 6. 调试优先级（遇到“没回复”类问题）

1. 看 iPad 是否在线：`/api/ipad-callback/status`
2. 看消息是否入库：`poll_and_save` 日志 + `wecom_message`/`conversation` 记录
3. 看 AI 是否生成：`ai_service` 日志（重试/降级/过滤）
4. 看是否被过滤：`BANNED_PHRASES` 命中情况
5. 看发送是否成功：`send_text` 调用返回值

## 7. 文档同步规则

- 任意功能级改动必须同步更新至少一个文档：
  - 架构变化 -> `ARCHITECTURE.md`
  - 进度/问题变化 -> `HANDOVER.md`
  - 部署或使用变化 -> `README.md`

## 8. 输出风格（给未来 AI）

- 结论先行：先说“改了什么、影响什么、怎么验证”。
- 使用明确路径引用（如 `backend/app/services/ai_service.py`）。
- 给出可执行命令，不给空泛建议。
# 乔门智脑 AI 销售系统 — Cursor AI 规则

## 项目身份

这是「乔门智脑」—— 一套企业微信 AI 销售自动化系统。
- 后端：Python 3.13 + FastAPI + SQLAlchemy async + SQLite（可切换 PostgreSQL）
- 前端：React 19 + TypeScript + Ant Design + Zustand + Vite
- AI：OpenAI 兼容 API（可配置中转），支持工作流、RAG、AI 教练
- WeChat 接入：JuheBot iPad 协议（GuidRequest API）
- 项目路径：`e:\xiaoshouzhinao\`
- 生产服务器：`106.14.41.135`，路径 `/opt/xiaoshouzhinao`，域名 `tfjzu.top`

---

## 一、代码风格规则

### Python 后端

- **异步优先**：所有数据库操作、HTTP 请求、文件 IO 必须使用 `async/await`
- **类型注解**：所有函数参数和返回值必须有类型注解，使用 Python 3.10+ 联合类型语法（`str | None`，不用 `Optional[str]`）
- **日志规范**：每个模块顶部 `logger = logging.getLogger("qiaomen.<模块名>")`，使用 `logger.info/warning/error`，禁止 `print()`
- **SQLAlchemy**：始终使用 `async with async_session_factory() as db:` 创建会话，查询用 `select()` + `await db.execute()`，不用同步 ORM
- **Pydantic**：所有 API 请求/响应用 `BaseModel`，禁止直接返回 ORM 对象
- **错误处理**：服务层用 try/except 捕获异常并记录日志，路由层用 `HTTPException` 返回规范错误
- **配置统一**：所有配置从 `from app.config import settings` 读取，禁止在代码中硬编码 URL/密钥

### TypeScript 前端

- **严格类型**：所有变量、函数参数、API 响应必须有 TypeScript 类型（`interface` 或 `type`）
- **API 调用**：必须通过 `frontend/src/api/` 下对应模块调用，禁止在组件内直接用 `axios`
- **状态管理**：业务状态放 Zustand store（`stores/`），组件局部状态用 `useState`
- **组件规范**：React 函数组件 + hooks，禁止 class component
- **样式规范**：优先用 Ant Design 组件属性，次选 inline `style={{}}`，不引入 Tailwind/其他 CSS 框架
- **国际化**：UI 文字全部用中文，不做 i18n，Ant Design `ConfigProvider` 已配置 `zhCN`

---

## 二、架构约定（必须遵守）

### 后端分层

```
Router（路由）→ Service（业务逻辑）→ Model（数据库）
```

- **Router 只做**：参数校验、调用 Service、返回响应，禁止在 Router 写业务逻辑
- **Service 只做**：业务逻辑、调用其他 Service 或 Model，不直接处理 HTTP 请求
- **Model 只做**：ORM 字段定义，不放业务逻辑

### 前端分层

```
Page（页面）→ API（接口调用）→ Store（状态）→ Component（展示）
```

- **所有后端接口调用**必须在 `frontend/src/api/` 对应文件中定义
- **新增页面**：在 `App.tsx` 加懒加载路由，在 `MainLayout.tsx` 加侧边栏菜单项

### 新增功能清单

新增一个完整功能需要：
1. `backend/app/models/` 新建或修改 ORM 模型
2. `backend/app/routers/` 新建或修改路由
3. `backend/app/services/` 新建或修改业务逻辑
4. `backend/app/schemas/` 新增 Pydantic schema
5. `backend/app/main.py` 注册路由（`app.include_router`）
6. `frontend/src/api/` 新增 API 方法
7. `frontend/src/pages/` 新增或修改页面
8. `frontend/src/App.tsx` 新增路由（如需要）
9. `frontend/src/layouts/MainLayout.tsx` 新增菜单（如需要）

---

## 三、核心业务逻辑（必须了解）

### iPad 协议消息流（最核心）

```
JuheBot API (/msg/sync_msg)
  → WecomIpadService.poll_and_save()   [每5秒轮询]
  → 过滤：content_type in (0,2)，sender/receiver 含 "788"
  → 保存 WecomMessage + Contact + Conversation
  → _handle_auto_reply(contact_id, staff_id)
  → per-contact asyncio.Lock（防并发）
  → 8秒去重窗口
  → ai_service.generate_reply()
  → BANNED_PHRASES 过滤（只过滤 AI 身份暴露词，不过滤正常客服词）
  → WecomIpadService.send_text()
```

**关键注意**：
- `BANNED_PHRASES` 列表在 `backend/app/services/ai_service.py` 中，**只应包含 AI 身份暴露词**（如"作为AI"、"I'm Claude"），**绝对不能**包含「抱歉」「感谢」等正常客服词，否则 AI 回复会被误杀
- JuheBot API 统一入口：`POST https://chat-api.juhebot.com/open/GuidRequest`，所有操作通过 `path` 参数区分

### AI 回复生成（ai_service.py）

- 主模型：`settings.AI_MODEL`（如 gpt-4o）
- 失败降级：3次指数退避（2s/5s/12s）→ 降级到 `settings.AI_MODEL_LITE`
- 限速：滑动窗口，每分钟最多 `min(RATE_LIMIT_PER_MINUTE, 50)` 次
- 响应缓存：相同上下文（最近3条消息）的请求，5分钟内复用缓存

### AI 工作流执行（workflow_engine.py）

工作流在客户消息到达时通过 `wecom.py` 的 `_try_execute_workflow()` 触发。节点类型：
- `intent` — AI 意图识别，匹配分支
- `condition` — 条件判断（客户属性/变量）
- `llm` — 调用 AI 生成内容
- `variable` — 设置/读取变量（存在 `assignments[]` 中）
- `script` — 返回固定话术模板
- `multimedia` — 发送图片/文件
- `end` — 结束节点

### 企微 AI 模式（4种）

在 `wecom_store.ts` 和数字员工配置中，AI 有 4 种工作模式：
- `auto` — 全自动回复，AI 自动发送
- `reference` — 参考模式，AI 给建议，销售手动决定是否发
- `collab` — 协作模式，AI 参与对话但需销售确认
- `off` — 关闭 AI，纯人工

---

## 四、数据库约定

- **ORM 文件位置**：`backend/app/models/` 每个实体一个文件
- **所有 model 继承**：`from sqlalchemy.orm import DeclarativeBase` 的 `Base`
- **时间字段**：统一用 `datetime.utcnow`，时区处理注意 JuheBot 返回的 `sendtime` 是 UTC+8 epoch seconds
- **外键关系**：用 `ForeignKey` + `relationship`，懒加载 `lazy="selectin"`（async 兼容）
- **当前无迁移**：`init_db()` 直接 `create_all`，生产切换时需先建 Alembic 迁移脚本

---

## 五、部署约定

- **环境变量**：所有密钥在 `backend/.env` 中，**绝对不提交 .env 到 Git**
- **Docker**：开发用 `docker-compose.yml`（backend host网络），生产用 `docker-compose.prod.yml`（bridge网络+TLS）
- **代码部署**：`python upload_project.py` 上传文件，然后服务器上执行 `docker compose up -d --build`
- **日志查看**：`docker compose logs -f backend --tail=100`
- **后端启动日志名称**：`qiaomen`（`logging.getLogger("qiaomen")`）

---

## 六、禁止事项

1. **禁止在代码中硬编码** API 密钥、服务器 IP、数据库密码
2. **禁止提交** `.env` 文件
3. **禁止在 BANNED_PHRASES 中添加** 「抱歉」「感谢」「您好」等正常客服词（历史教训：曾因此导致 AI 全量回复被屏蔽）
4. **禁止在路由层写业务逻辑**（保持路由薄，逻辑在 Service）
5. **禁止在前端组件内直接 axios 调用**（必须通过 `api/` 模块）
6. **禁止在 wecom_ipad_service.py 的 _handle_auto_reply 中移除 per-contact Lock**（会导致同一客户并发发多条重复回复）
7. **禁止同步数据库调用**（SQLite/PostgreSQL 均为 async，同步调用会阻塞事件循环）

---

## 七、常用调试方法

```bash
# 查看 iPad 消息轮询日志
docker compose logs -f backend | grep "iPad poll"

# 查看 AI 自动回复日志
docker compose logs -f backend | grep "auto-reply"

# 检查 BANNED_PHRASES 是否误杀回复
docker compose logs -f backend | grep "BANNED"

# 手动检查 iPad 在线状态
curl -H "Authorization: Bearer <token>" http://localhost:8000/api/ipad-callback/status

# 查看数据库（SQLite）
docker exec -it xiaoshouzhinao-backend-1 python -c "
import asyncio, sqlite3
conn = sqlite3.connect('/app/qiaomen.db')
rows = conn.execute('SELECT id, content, created_at FROM wecom_messages ORDER BY id DESC LIMIT 5').fetchall()
for r in rows: print(r)
"
```

---

## 八、关键文件速查

| 文件 | 用途 |
|------|------|
| `backend/app/main.py` | 应用入口，后台任务注册 |
| `backend/app/config.py` | 所有配置项定义 |
| `backend/app/services/ai_service.py` | AI 调用、BANNED_PHRASES、重试降级 |
| `backend/app/services/wecom_ipad_service.py` | JuheBot iPad 协议，消息收发 |
| `backend/app/services/workflow_engine.py` | 工作流节点执行引擎 |
| `backend/app/routers/wecom.py` | 企微聊天 API，AI 教练，工作流触发 |
| `backend/app/routers/ipad_callback.py` | iPad 状态检查、QR 码登录 |
| `frontend/src/App.tsx` | 路由定义 |
| `frontend/src/layouts/MainLayout.tsx` | 侧边栏菜单 |
| `frontend/src/pages/sales/components/WecomChatPanel.tsx` | 聊天面板（最复杂的前端组件）|
| `frontend/src/api/wecom.ts` | 企微相关 API 封装 |
| `frontend/src/stores/wecomStore.ts` | 企微工作台状态 |
| `docker-compose.yml` | 开发环境 Docker 配置 |
| `backend/.env.example` | 环境变量模板 |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frederderrickkastre-arch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
