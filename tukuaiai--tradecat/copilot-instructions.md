## tradecat

> 本文件面向自动化/AI 编码 Agent：**给出可执行路径 + 明确边界约束**，避免“大改/改出事故”。

# TradeCat - AI Agent 操作手册（Operating Manual）

本文件面向自动化/AI 编码 Agent：**给出可执行路径 + 明确边界约束**，避免“大改/改出事故”。

## 核心

以数据表↔对象映射为中心来组织系统
真正长期维护的资产不是采集器，而是 dataset、schema、语义和交付契约
统一模板、公共基类/契约、CI 守护
新增能力通常是新增 dataset，而不是在旧流程里继续打补丁  
边界清晰：一个 dataset 就是一个稳定责任单元，天然适合测试、回补、血缘、权限、SLA。
扩展自然：新增能力通常是新增 dataset，而不是在旧流程里继续打补丁。
架构收敛：schema-first 能把“先写代码再补表结构”的混乱扼杀掉。
核心领域模型应面向对象，但 ETL/校验/转换链路不必强行 OOP，函数式流水线往往更直接。

---

## 1) Mission & Scope（目标与边界）

### 允许

- 修改服务代码：`core/**/src/`、`plugins/**/src/`
- 修改服务脚本：`core/**/scripts/`、`plugins/**/scripts/` 与 `scripts/`
- 更新文档：`README.md`、`README_EN.md`、`AGENTS.md`、`assets/docs/**`
- 更新配置模板：`assets/config/.env.example`

### 禁止

- 禁止修改生产配置：`assets/config/.env`（包含密钥/凭证）（除非任务明确要求）
- 禁止删除或改写数据库文件：`assets/database/**`（除非任务明确要求）
- 禁止大范围重构（无明确任务授权）
- 禁止引入未经验证的第三方依赖（新增依赖必须同时更新锁文件/说明）

### 敏感区域（只读/慎动）

| 路径 | 说明 |
|:---|:---|
| `assets/config/.env` | 运行时私密配置（不提交） |
| `assets/database/services/**` | SQLite 持久化与审计数据 |
| `backups/` | 导出/备份产物 |

---

## 2) Golden Path（推荐执行路径）

```bash
# 0) 环境检查（推荐）
./scripts/check_env.sh

# 1) 初始化（创建各服务 .venv + 安装依赖）
./scripts/init.sh

# 2) 准备配置（模板 → 运行时）
cp assets/config/.env.example assets/config/.env && chmod 600 assets/config/.env
vim assets/config/.env

# 3) 启动核心服务
# ⚠️ 重要：consumption 层禁止直连数据库；顶层 start.sh 默认包含 Query Service（api-service，/api/v1），并保证其先于 telegram/sheets 启动。
./scripts/start.sh start
./scripts/start.sh status

# （可选）冒烟检查（不回显 token）
./scripts/smoke_query_service.sh

# 4) 修改代码后验证
./scripts/verify.sh
```

---

## 3) Must-Run Commands（必须执行/常用）

### 顶层脚本（来源：`scripts/`）

- 初始化：`./scripts/init.sh`
- 启动/停止/状态：`./scripts/start.sh start|stop|status|restart`
- 守护模式：`./scripts/start.sh daemon|daemon-stop`
- 校验：`./scripts/verify.sh`

### 单服务（来源：各服务 `Makefile`）

```bash
cd core/<domain>/<service-name>       # domain: market/alternative/storage
# Query Service（统一读出口）：
cd core/query
# 或：
cd plugins/<service-name>
make install
make lint
make format
make test
make start|stop|status
```

---

## 4) Code Change Rules（修改约束）

### 架构分层（单向依赖）

```text
core (market/alternative) -> TimescaleDB (LF/HF) -> plugins(派生: trading/signal) -> PG(indicator_snapshots.* / signal_runtime.*) -> core/query (/api/v1) -> plugins(导出/网关/展示)
```

- core/market、core/alternative：只做“事实采集/标准化/写入”（不做派生计算）
- plugins（如 `plugins/trading` / `plugins/signal`）：只读事实库，计算派生结果并写入 PostgreSQL 派生库（`indicator_snapshots.*` / `signal_runtime.*`）
- core/query（query；兼容旧名 api-service）：**唯一读出口**（多数据源 DSN）；对外提供 `/api/v1/*`
- plugins（如 `plugins/telegram` / `plugins/sheets` / `plugins/llm-gateway` / `plugins/vis`）：通过 core/query HTTP 读取数据并导出/展示（Sheets/Telegram/LLM 等；**禁止直连 DB**）
- `plugins/ai`：研究型 / 离线分析插件；当前允许只读直连 `facts_data` / `derived_data`（架构例外），但不是新的对外读出口，也不得写库

### 服务边界（来自仓库结构 `core/` + `plugins/`）

- `core/market/*`：行情事实采集（写 TimescaleDB）
- `core/alternative/*`：另类事实采集（写入事件/非结构化事实库）
- `plugins/trading`、`plugins/signal`：派生计算（写 `indicator_snapshots.*`、`signal_runtime.*` 等派生库）
- `core/query/*`：统一读出口（/api/v1）
- `plugins/telegram`、`plugins/sheets`：导出/投递（Telegram/Sheets）
- `plugins/llm-gateway`：LLM/nofx 网关
- `plugins/predict`、`plugins/vis`、`plugins/fate`、`plugins/ai`、`plugins/nofx-dev`：其它插件/实验（以目录内说明为准）

### 兼容性要求（以仓库为准）

- 推荐 Python：**3.12+**
  - 证据：CI 使用 Python 3.12（`.github/workflows/ci.yml`）
  - 证据：各服务 `core/**/pyproject.toml` / `plugins/**/pyproject.toml` 声明 `requires-python = ">=3.12"`
- 脚本最低门槛：`scripts/init.sh` 检查 Python >= 3.10（不代表所有服务在 3.10 下可用）
- 数据库端口（模板默认，来源：`assets/config/.env.example`）：
  - 默认单端口：`FACTS_DATABASE_URL` 指向 `localhost:5433/facts_data`，`DERIVED_DATABASE_URL` 指向 `localhost:5433/derived_data`
  - 兼容旧服务：`DATABASE_URL` 默认仍指向 `facts_data`
  - `assets/contracts/resources.v1.yaml` / `probes.v1.yaml` / `bindings.v1.yaml` 是中央契约唯一真相源；`assets/catalog/*` 只是导出 projection
  - `assets/config/.env.example` 只保留连接、鉴权、feature flag 与少量未迁入 central contracts 的动态对象名
  - 共享实现入口：`assets/common/contracts/db_contracts.py`；服务应通过 central contracts 派生读取，避免业务代码裸写对象名
  - 若拆分 LF/HF 双集群：把 `BINANCE_HF_DATABASE_URL` 指向另一实例/端口；LF/HF 只是运行分组，不再是子工程目录
  - Query Service（消费层必需）：`QUERY_SERVICE_BASE_URL` 默认 `http://127.0.0.1:8088`（可选 `QUERY_SERVICE_TOKEN` 开启内网鉴权）
  - `facts_data` 当前事实层推荐路径：
    - `market.*`：事实行情主路径
    - `alternative.*`：事实另类主路径
    - `public` / `_timescaledb_*` / `timescaledb_*`：系统/扩展层，不作为业务主路径

### 依赖添加规则

1) 先确认仓库/服务内是否已存在相同依赖
2) 只在对应服务的 `requirements*.txt` 或 `pyproject.toml` 增加
3) 如仓库要求锁定：同步更新 `requirements.lock.txt`（服务内已有）
4) 文档同步：`assets/config/.env.example` / README / AGENTS 需要同步说明

---

## 5) Style & Quality（质量标准）

- Python 格式化/静态检查：ruff（服务内 `pyproject.toml` 配置）
- 行长：120（服务内 ruff 配置）
- 日志：使用 `logging`，异常必须 `exc_info=True`（避免吞栈）

---

## 6) Project Map（结构速览）

```text
tradecat/
├── assets/
│   ├── common/                # 共享工具库（`import assets.*`）
│   ├── alternative/           # alternative 公共资产（schema/docs/external/skills 等）
│   ├── config/                # 配置模板/运行时 .env（不提交）
│   ├── docs/                  # 项目文档（mkdocs 入口；`index.md` 为总入口，`core/` 存放仓库核心文档）
│   ├── archive/               # 历史归档源码/文档（不属于主链）
│   ├── tasks/                 # 任务文档
│   ├── artifacts/             # 构建/分析产物（默认忽略）
│   ├── database/              # DDL/CSV/SQLite（敏感：勿改写持久化数据）
│   ├── repo/                  # 外部仓库镜像（默认忽略）
│   └── tests/                 # 资产/SQL/脚本级测试素材
├── scripts/
│   ├── alternative/           # alternative 统一管理脚本（manage.sh 等）
│   ├── init.sh
│   ├── start.sh
│   ├── check_env.sh
│   └── verify.sh
├── core/
│   ├── market/
│   │   ├── binance/                 # Binance 单工程根：src/binance + var/{lf,hf}
│   │   └── hyperliquid/            # 占位：未来扩展数据源
│   ├── alternative/                # 另类事实采集（多源采集集群）
│   │   ├── api/                     # 注意：这是 alternative 集群的 API（不要与 Query Service 混淆）
│   │   ├── news/
│   │   ├── telegram/                 # TG 主采集链（历史辅服务已归档）
│   │   ├── discord/
│   │   ├── x/
│   │   ├── investing/
│   │   ├── hyper/
│   │   └── ...
│   ├── query/                      # 统一读出口（Query Service，/api/v1；服务根目录）
│   └── storage/                   # DDL/迁移/保留策略（预留）
├── plugins/
│   ├── trading/                   # 派生：指标计算
│   ├── signal/                    # 派生：信号检测
│   ├── ai/                        # AI：就绪检查/离线分析入口（非常驻进程）
│   ├── telegram/                  # 导出：Telegram Bot + 事件/信号推送
│   ├── sheets/                    # 导出：Google Sheets 同步
│   ├── llm-gateway/               # 网关：OpenAI compatible（nofx 接入）
│   ├── predict/                   # 其它：预测市场
│   ├── fate/                      # 其它：命理/排盘
│   ├── vis/                       # 其它：可视化
│   └── nofx-dev/                  # 其它：外部工程镜像
└── assets/                        # 共享资产根：common/database/repo/tests 等
```

---

## 7) Common Pitfalls（常见坑与修复）

### `.env` 权限导致服务启动失败

```bash
chmod 600 assets/config/.env
```

### TimescaleDB 端口不一致

- 证据：`assets/config/.env.example` 默认单端口 5433（LF/HF 同库同端口）
- 修复：统一 `assets/config/.env` 中的端口，并同步所有脚本/示例命令

### TA-Lib 安装失败（形态指标需要）

```bash
sudo apt-get update
sudo apt-get install -y build-essential

wget http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz
tar -xzf ta-lib-0.4.0-src.tar.gz
cd ta-lib && ./configure --prefix=/usr && make && sudo make install
cd .. && rm -rf ta-lib ta-lib-0.4.0-src.tar.gz

pip install TA-Lib
```

### sheets-service 在代理环境下偶发 SSL 抖动

现象（常见日志）：`SSLError: DECRYPTION_FAILED_OR_BAD_RECORD_MAC`

- 若使用 SA 写入：可设置 `SHEETS_SA_NET_WRITE_RETRIES` 提升幂等写入的重试容忍度
  - 证据：`plugins/sheets/src/sa_sheets_writer.py` 读取该环境变量

---

## 8) PR / Commit Rules（提交与 CI）

- Commit message 推荐：

```text
<type>(<scope>): <subject>
```

`type`: `feat|fix|docs|refactor|chore|style`

- CI（证据：`.github/workflows/ci.yml`）：
  - ruff check（针对 `core/` + `plugins/`）
  - py_compile 抽样（前 50 个 `.py`）

---

## 9) Documentation Sync Rule（强制同步规则）

当你修改了以下任一项，必须同步更新文档：

- 新增/修改命令：更新 `README.md` / `AGENTS.md`
- 新增/修改配置项：更新 `assets/config/.env.example`，并在 `README.md` / `AGENTS.md` 说明
- 目录结构/服务职责变更：更新 `README.md` / `AGENTS.md`

不确定项必须写 `TODO` 并标注需要核对的文件路径/字段名，禁止猜测。

---
> Source: [tukuaiai/tradecat](https://github.com/tukuaiai/tradecat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
