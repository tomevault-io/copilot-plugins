## bgg-master

> > 本文档供代码 Agent、开发人员和运维人员共同使用。它定义工作方式、架构边界、业务不变量、验证门槛和发布流程；项目方向和未完成任务以 [`PROJECT_PLAN.md`](PROJECT_PLAN.md) 为准。

# BGG Agent、开发与运维指南

> 适用范围：本仓库及其子目录。
>
> 最后整理：2026-07-10。
>
> 本文档供代码 Agent、开发人员和运维人员共同使用。它定义工作方式、架构边界、业务不变量、验证门槛和发布流程；项目方向和未完成任务以 [`PROJECT_PLAN.md`](PROJECT_PLAN.md) 为准。

## 1. 信息来源与优先级

遇到描述不一致时，按以下顺序判断：

1. 当前代码、Alembic migration、实际配置校验和运行结果。
2. 本文档中的仓库规则与业务不变量。
3. `PROJECT_PLAN.md` 中的当前风险、优先级和任务。
4. `README.md`、`README.zh-CN.md` 和 `deploy/DEPLOY_LAN.md` 中的使用说明。
5. Git 历史仅用于追溯，不作为当前行为的直接依据。

上级目录中的 `report_board`、`Post` 和 `chat_bot` 文档只提供工程经验，不是 BGG 的配置来源。不要直接复制它们的端口、密钥、数据模型、命令或权限规则。

`BGG_frontend/` 只作为前端开发时的页面结构、视觉样式、交互效果、文案和图片素材参考，不是当前应用的正式代码或配置来源。可以选择性地把其中合适的前端表现迁入正式 `frontend/`，但不得复制或依赖其中基于 `localStorage` 的业务数据、模拟 API、认证、路由、数据结构或运行配置。所有关键设置、后端行为、数据库模型、Alembic migration、权限规则、API 契约、部署方式和业务事实均以本 Git 仓库中的当前正式代码、配置及实际运行结果为准。

## 2. 接手任务时的固定流程

### 2.1 开始前

- 确认当前目录是 `/home/merrill/report-board/BGG`。
- 执行 `git status --short --branch`，区分已有修改和本次修改。
- 阅读 `PROJECT_PLAN.md`，确认任务优先级及已知阻塞。
- 只打开与任务有关的代码、migration 和文档；行为以真实代码为准。
- 检查 `.env`、数据库和上传目录是否属于生产或真实业务数据。
- 若任务会改变 schema、删除数据、发邮件、开放端口或重启服务，先明确备份和影响范围。

### 2.2 实施中

- 保留用户已有的未提交修改，不覆盖、不回滚、不顺手格式化无关文件。
- 先修根因，再补兼容；临时兼容必须写明删除条件。
- 路由负责 API 边界，Pydantic model 负责输入输出，service 负责可复用业务事务。
- 写操作的权限和业务校验必须由后端执行，前端隐藏按钮不能代替鉴权。
- 修改数据库模型时同步创建 Alembic migration，不在启动时隐式修改 schema。
- 修改前端功能时同步检查韩文、中文和英文文案。
- 修改高风险行为时先定义失败、取消、并发和重试语义。

### 2.3 结束前

- 运行与风险相匹配的检查，记录实际命令和结果。
- 执行 `git diff --check`，检查意外文件和敏感信息。
- 更新应受影响的文档，但不要创建重复计划或完成报告。
- 交付说明包含：改了什么、验证了什么、还剩什么、是否需要迁移或运维动作。

## 3. 项目地图

```text
BGG/
├── AGENTS.md                 # Agent、开发和运维共同规则
├── PROJECT_PLAN.md           # 方向、风险、阶段和未完成任务
├── README.md                 # 韩文使用入口、API 和配置概览
├── README.zh-CN.md           # 中文使用入口、API 和配置概览
├── .env.example              # 可提交的配置模板，不含真实密钥
├── run.sh                    # 建环境、执行迁移、启动 8100
├── deploy/
│   ├── bgg.service           # systemd 服务定义
│   └── DEPLOY_LAN.md         # 局域网运行手册
├── backend/
│   ├── main.py               # FastAPI 入口、路由和静态前端挂载
│   ├── app/
│   │   ├── config.py         # 环境变量和生产配置保护
│   │   ├── database.py       # async engine、session、SQLite WAL
│   │   ├── dependencies.py   # 管理端鉴权依赖
│   │   ├── orm_models.py     # SQLAlchemy ORM
│   │   ├── models/           # Pydantic schema
│   │   ├── routers/          # API 路由
│   │   └── services/         # 预约、SSE、邮件等业务逻辑
│   └── alembic/versions/     # schema 变更的唯一迁移历史
├── frontend/
│   ├── *.html                # 原生页面
│   ├── js/                   # 页面逻辑、API 调用、i18n
│   ├── css/                  # 公共和页面样式
│   └── img/                  # 版本化静态图片
└── uploads/                  # 运行期上传文件，不提交内容
```

## 4. 运行架构

BGG 是 FastAPI 直接服务原生 HTML/CSS/JS 的同源应用：

```text
Browser
  ├── /, /*.html, /css, /js, /img
  ├── /api/* --------------------> FastAPI routers
  ├── /api/sse ------------------> SSE client registry
  └── /uploads/* ----------------> uploaded files

FastAPI routers
  ├── Pydantic validation
  ├── require_auth for admin writes
  ├── SQLAlchemy AsyncSession
  └── services for transaction/mail/SSE

Database
  ├── SQLite: development and single-host verification
  └── PostgreSQL: intended production database
```

API 路由必须在通配页面路由之前挂载。新增页面路由时不得吞掉 `/api/*`、`/uploads/*` 或静态资源请求。

## 5. 核心业务不变量

### 5.1 医院与内容

- 医院 ID 是数据库主键，不使用数组位置或页面顺序作为身份。
- 详情页、预约配置、封锁时段、促销和评价通过医院 ID 关联。
- 前端业务数据以 API 为准，不重新引入 localStorage 作为事实来源。
- localStorage 只允许保存语言、活动标签等纯 UI 偏好。
- 删除医院前必须明确关联预约、内容和上传文件的保留策略。

### 5.2 预约与时段

- 同一医院、日期和时间只能有一个 `blocked_slots` 记录，数据库唯一约束是最终并发防线。
- 创建预约和写入 `reason=reservation` 的封锁时段必须在同一事务中完成。
- 冲突必须返回 `409`，不能静默覆盖或创建双重预约。
- 取消预约时只释放该预约产生的封锁时段，不删除管理员手动封锁。
- 改动预约或时段后应通知对应医院的 SSE 客户端刷新可用性。
- 用户查询接口只返回完成查询所需的最少字段，不暴露后台备注或其他用户资料。

### 5.3 CSV 和批量操作

- “预览”必须是无副作用操作；用户确认前不得修改数据库。
- “替换”和“合并”必须有明确、可测试的服务端语义。
- 取消、解析失败或保存失败时，原数据必须保持不变。
- 批量导入需要限制大小、编码和字段，并报告跳过或失败的记录。

当前 CSV 预览仍违反以上规则，属于 `PROJECT_PLAN.md` 的 `P0-02`，不要基于现状继续扩展。

### 5.4 认证与权限

- 目标状态是 Cookie-only 管理认证：JWT 存 HttpOnly Cookie，不返回给 JavaScript。
- 所有管理端写接口使用统一后端鉴权依赖。
- 公开接口只开放预约提交、最小化预约查询、公开内容读取和 SSE 等必要能力。
- 登录失败不能泄漏密码、token、内部路径或配置。
- `ENV=production` 下弱默认密钥和默认管理员密码必须阻止启动。

当前 Bearer/sessionStorage 兼容路径属于 `PROJECT_PLAN.md` 的 `P0-04`，新代码不得继续依赖它。

### 5.5 邮件与外部服务

- API 返回成功不等于邮件已经投递；界面文案必须区分保存、排队、成功和失败。
- SMTP 密码、Google Maps key 等第三方凭据只放环境变量或服务端安全配置。
- 外部服务失败不能破坏已提交的核心业务事务。
- 用户姓名、电话、邮箱、咨询和预约内容属于敏感数据，不写入普通调试日志。

## 6. 后端开发规则

- Python 使用 4 空格、snake_case、类型标注和 SQLAlchemy 2.x 风格。
- 路由只做参数、权限和响应边界；跨路由复用逻辑放入 `backend/app/services/`。
- API 输入输出放入 `backend/app/models/`，避免直接把任意 dict 当作长期契约。
- 状态字段使用明确允许值，不接受任意字符串。
- 事务中任何异常都应 rollback，再返回稳定的 HTTP 错误。
- 不在异常响应中返回数据库 SQL、绝对路径、密钥或 traceback。
- SQLite 和 PostgreSQL 都要支持的字段与查询，不使用仅单一数据库可用的隐式行为。
- 时间字段新增或重构前先定义时区策略；不要继续混用不明确的 naive UTC、本地时间和字符串日期。

## 7. 数据库与迁移规则

- `backend/app/orm_models.py` 是当前 ORM，`backend/alembic/versions/` 是 schema 历史。
- 任何新增表、字段、索引、唯一约束或类型变化都必须有 migration。
- migration 应支持空库升级和已有库升级，并尽量可重复验证。
- 不删除或改写已发布 migration；用新的 revision 修正。
- 应用启动不自动 `create_all()` 或修改生产 schema。
- 发布前备份数据库和上传目录，迁移后检查当前 revision 和关键数据量。
- SQLite 文件复制备份时应确保一致性，不能忽略活动中的 WAL；优先在维护窗口停止写入或使用 SQLite backup 方法。
- PostgreSQL 使用数据库原生备份工具，并实际验证恢复。

已知问题：当前环境中 `alembic current` 和 `alembic upgrade head` 可能不退出，见 `P0-01`。在该问题解决前，不把迁移超时误报为成功，也不绕过迁移直接发布。

## 8. 前端开发规则

- 继续使用原生 HTML/CSS/JS，不为单个需求引入前端框架或构建链。
- 同源 API 基址由 `frontend/js/common.js` 的 `API_BASE` 约定管理。
- 公开读取优先复用公共 helper；管理端请求统一经过 `adminFetch()`，不要重复实现 401 和认证处理。
- 动态数据写入 `innerHTML` 前必须调用统一转义函数；URL 还需限制允许协议和来源。
- 新代码使用 `addEventListener()`，不要继续增加 HTML 内联事件属性。现有内联事件应按专项任务逐步迁移，不在无关任务中大面积改写。
- 每个异步操作都要有 loading、成功、空状态和失败反馈，保存失败不能表现成成功。
- 缓存医院、预约配置或封锁时段时，写操作和 SSE 事件后必须失效对应缓存。
- 韩文、中文、英文 key 同时维护；新增 key 后检查所有语言和默认回退。
- 修改页面结构时检查对应 JS 使用的静态元素 ID；修改 CSS/资源时检查所有 HTML 引用存在。
- 图片和文本的用户输入不得直接拼接为可执行 HTML 或事件处理器。

## 9. 安全与数据处理

- `.env`、数据库、WAL 文件、上传内容、日志和真实服务器参数不提交 Git。
- 命令输出和交付说明不得显示 JWT、管理员密码、SMTP 密码、API key 或完整用户资料。
- `.env.example` 只放占位值，并与 `config.py` 支持的变量保持同步。
- 上传校验不能只信任浏览器声明的 MIME；正式发布前必须验证文件头和实际图片解码。
- 上传文件使用服务端生成的安全名称，不把原始文件名当路径。
- 写接口的 CORS、Cookie、CSRF 和限流策略应作为一个整体审查。
- 不直接在公网暴露 Uvicorn；生产经 Nginx/HTTPS，应用只监听回环或受控内网地址。
- 涉及个人数据导出、保留、删除或日志采集时，先确认产品和合规口径。

## 10. 验证命令与质量门槛

### 10.1 当前可用的基础检查

从仓库根目录执行：

```bash
for file in frontend/js/*.js; do node --check "$file" || exit 1; done
backend/.venv/bin/python -m compileall -q backend/app backend/main.py
git diff --check
```

检查数据库和迁移时从 `backend/` 执行，但要注意 `P0-01`：

```bash
cd backend
.venv/bin/python -m alembic current
.venv/bin/python -m alembic check
.venv/bin/python -m alembic upgrade head
```

仓库目前没有正式自动化测试套件。新增测试后统一使用 `pytest`，测试数据库和上传目录必须与真实运行数据隔离。

### 10.2 按改动类型验证

| 改动 | 最低验证 |
|---|---|
| Python 路由/model/service | Python 编译、相关 API 成功/失败/权限路径 |
| ORM 或 migration | 空库升级、已有库升级、当前 revision、关键查询 |
| 预约或时段 | 正常预约、并发冲突、取消释放、管理员封锁、SSE 刷新 |
| 认证 | 登录成功/失败、刷新、过期、退出、未授权写请求 |
| 上传 | 合法图片、伪造 MIME、超限文件、未授权、路径安全 |
| CSV/批量导入 | 预览零写入、取消、合并、替换、非法文件、失败回滚 |
| 前端 JS | 全部 `node --check`、浏览器成功/失败/空状态 |
| HTML/CSS/i18n | 资源无坏链、静态 ID 兼容、三语言、移动端检查 |
| 部署 | migration、服务重启、health、日志、关键浏览器流程、回滚准备 |

高风险修改不能只验证 happy path。数据库、认证、预约冲突、上传和邮件至少覆盖一个失败路径。

## 11. 本机与同机项目边界

根据上级项目文档，当前同一服务器存在多个服务：

| 服务 | 已知用途 | 端口/入口约定 |
|---|---|---|
| `report_board` | 内部报告与工作流 | 直接应用端口 `8000`，通常由 Nginx 统一入口 |
| `BGG` | 医疗旅游平台 | 默认 `8100` |
| `Post` | 海报系统 | 后端约 `3100`，正式入口倾向 `/poster` 反代 |
| Nginx | 统一 HTTP/HTTPS 入口 | `80/443` |

这些值可能随部署变化，操作前用 `ss`、systemd 和 Nginx 配置核实。不要抢占 `8000`、`3100`，不要为方便测试永久开放新的高端口。跨项目接入优先使用明确的 URL、签名 SSO 或 API 契约，不共享数据库文件、登录密钥或进程内状态。

## 12. 开发运行

首次配置：

```bash
cp .env.example .env
# 设置 JWT_SECRET、ADMIN_PASSWORD；局域网 HTTP 保持 COOKIE_SECURE=false
```

一键运行：

```bash
./run.sh
```

手动运行：

```bash
cd backend
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/python -m alembic upgrade head
.venv/bin/uvicorn main:app --host 0.0.0.0 --port 8100
```

访问：

- 前台：`http://127.0.0.1:8100/`
- 后台：`http://127.0.0.1:8100/admin.html`
- API：`http://127.0.0.1:8100/docs`
- 健康检查：`http://127.0.0.1:8100/api/health`

## 13. 运维与发布流程

### 13.1 发布前

- [ ] 明确发布版本、提交和变更范围。
- [ ] 确认没有未解释的工作区修改。
- [ ] 备份数据库和 `uploads/`，记录恢复位置。
- [ ] 核对 `.env` 必填项、权限和文件路径，不输出真实值。
- [ ] 确认 Alembic 单一 head，并在同类测试数据库验证升级。
- [ ] 完成与改动风险匹配的自动化和手动验证。
- [ ] 准备代码回滚和数据恢复步骤。

### 13.2 发布

systemd 操作以 [`deploy/DEPLOY_LAN.md`](deploy/DEPLOY_LAN.md) 为准。标准顺序是：

1. 进入正确项目目录并确认目标提交。
2. 安装锁定/声明的依赖。
3. 执行 migration，失败或超时立即停止。
4. 重启 `bgg` 服务。
5. 检查 systemd 状态和应用日志。
6. 运行健康检查和关键业务验收。

### 13.3 发布后验收

- [ ] `/api/health` 返回 `status=ok` 和预期环境。
- [ ] 前台、后台、静态资源和上传图片可访问。
- [ ] 管理员能登录和退出，未登录写请求被拒绝。
- [ ] 医院读取、预约提交、冲突响应、预约查询正常。
- [ ] 后台内容修改能反映到公开页面。
- [ ] SSE 更新、邮件日志/投递状态符合预期。
- [ ] 日志没有持续异常，磁盘和数据库状态正常。

### 13.4 回滚原则

- 代码回滚和数据回滚分开处理，不假设 `git checkout` 能恢复数据库。
- migration 不安全回退时，优先恢复发布前备份，而不是手工逆改生产表。
- 上传目录和数据库必须恢复到彼此一致的时间点。
- 回滚后重复健康检查和关键业务验收，并记录失败原因。

## 14. 日常运营检查

建议按实际运营频率执行：

### 每日

- [ ] 服务和健康检查正常。
- [ ] 日志无持续 5xx、数据库锁、邮件或上传异常。
- [ ] 磁盘空间和上传目录增长正常。
- [ ] 新预约、咨询和后台处理链路可用。

### 每周

- [ ] 最近备份存在且大小合理。
- [ ] 检查失败邮件、孤立上传文件和异常预约状态。
- [ ] 核查证书、域名或第三方配额预警。
- [ ] 更新 `PROJECT_PLAN.md`：删除已完成项，调整当前风险和下一批次。

### 定期演练

- [ ] 从备份恢复数据库和上传目录到隔离环境。
- [ ] 验证管理员登录、预约、内容、上传和查询。
- [ ] 验证 systemd 重启、服务器重启和 Nginx 反代恢复。

## 15. 文档分工

| 文档 | 负责内容 | 何时更新 |
|---|---|---|
| `AGENTS.md` | 仓库规则、架构边界、不变量、验证和运维流程 | 规则、架构或发布流程变化时 |
| `PROJECT_PLAN.md` | 当前风险、方向、优先级和未完成清单 | 任务完成或优先级变化时 |
| `README.md` | 韩文启动、结构、API 和配置概览 | 用户可见能力或启动方式变化时 |
| `README.zh-CN.md` | 中文启动、结构、API 和配置概览 | 与韩文 README 同步 |
| `deploy/DEPLOY_LAN.md` | 局域网实际操作命令 | 端口、systemd、路径或排障步骤变化时 |

不要新建重复的阶段报告、逐文件说明或历史完成清单。历史通过 Git 查询；真实 IP、域名、账号和备份位置可记录在已被 Git 忽略的 `deploy/service.local.md`，密钥仍只放 `.env` 或外部密钥系统。

## 16. Definition of Done

一项开发或运维任务只有满足以下条件才算完成：

- 需求行为、失败语义和边界已经明确并实现。
- 权限、数据一致性和兼容性风险已处理。
- 有与风险匹配的测试或可重复验证记录。
- migration、配置、部署或回滚影响已说明。
- 三语言、错误状态和移动端影响已检查（如适用）。
- 相关文档已同步，计划中的已完成项已删除。
- 没有泄漏密钥、用户数据或真实运维参数。
- 最终差异只包含本任务需要的文件，且可以独立回滚。

## 17. 从相邻项目吸收的工程经验

本指南参考了上级目录中其他项目的现行文档，并只吸收适用于 BGG 的部分：

- `report_board/docs/AGENTS.md` 与架构文档：采用“真实代码优先、业务逻辑集中、显式 migration、高风险改动完整测试、文档职责分离”的规则。
- `report_board` 的可靠性和下一步文档：采用“只保留未完成计划、发布前同时备份数据库和媒体、健康检查、恢复演练、按风险而非单一覆盖率验收”的方法。
- `Post/PROJECT_MAP.md`：采用“给维护者真实项目地图、按常见修改场景定位文件、接口或目录变化时同步地图”的思路；BGG 的精简地图维护在本文档第 3 节。
- `Post` 的服务器化和部署文档：采用“服务端数据为事实来源、导入先 dry-run、systemd 常驻、Nginx 统一入口、发布与回滚成对设计”的方法。
- `chat_bot` 的多诊所架构：采用“集团共享信息与诊所专属数据分层”的长期思路，但不直接套用其文件结构。

BGG 当前并不是安全意义上的多租户系统。`hospital_id` 目前用于内容和预约关联，没有租户管理员、医院级权限、行级隔离或跨医院审计。如果未来允许多家诊所分别登录维护数据，必须先设计 tenant、用户归属、授权范围和数据迁移，不能仅靠前端筛选医院，也不能复制一套项目目录给每家诊所。

## 18. 当前必须先处理的事项

开始新功能前先查看 `PROJECT_PLAN.md`。当前 P0 稳定化问题已进入基线，后续优先级以 `PROJECT_PLAN.md` 的“当前风险与优先级”和“下一执行批次”为准。

当前执行方向：

1. 先建立独立测试数据库和 fixture，确保自动化测试不读写正式 `bgg.db`。
2. 补齐认证、CSV、预约冲突和取消释放路径的 API 级回归测试。
3. 建立关键浏览器 E2E 链路，再汇总 Python、JavaScript、迁移和测试检查命令。
4. 自动化验证稳定后，再推进 PostgreSQL 迁移兼容性、局域网完整验收和公网发布准备。

---
> Source: [parkjaerang/BGG_master](https://github.com/parkjaerang/BGG_master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
