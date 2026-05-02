## bilibili-analyzer

> - `frontend/`：Vue 3 + Vite 前端（`src/views`、`src/components`、`src/api`、`src/store`、`src/router`）。

# 仓库协作指南

## 项目结构与模块组织
核心目录如下：
- `frontend/`：Vue 3 + Vite 前端（`src/views`、`src/components`、`src/api`、`src/store`、`src/router`）。
- `backend/`：FastAPI 后端（`app/api`、`app/services`、`app/models`、`app/etl`、`app/ml`、`app/core`）。
- `streaming/`：Spark 流处理任务（`spark_streaming.py`）。
- `docs/`：数据库初始化脚本（`docs/database.sql`）。

后端测试文件统一放在 `backend/tests/test_*.py`。数据分析链路采用 `DWD -> DWS`，实现位于 `backend/app/etl/`。

## 构建、测试与开发命令
常用命令如下：

```bash
# 后端
cd backend
pip install -r requirements.txt
cp .env.example .env
python main.py

# 前端
cd frontend
npm install
npm run dev
npm run build

# 基础依赖服务（Kafka/Redis/Spark）
docker compose up -d kafka redis spark-master spark-worker

# 后端测试脚本
cd backend
python tests/test_etl.py
python tests/test_crawl_service.py
python tests/test_websocket.py <room_id>

# Spark 流处理任务
cd streaming
spark-submit spark_streaming.py
```

## 代码风格与命名规范
- Python 遵循 PEP 8，4 空格缩进；函数/文件使用 `snake_case`，类使用 `PascalCase`。
- Vue 单文件组件使用 `PascalCase.vue`，按业务功能分组。
- API 与服务模块采用领域命名，如 `videos.py`、`crawl_service.py`。
- 前端样式遵循 `frontend/STYLE_RULE.md`，复用 `frontend/src/styles/common.css` 变量，优先使用 `<style scoped>`。
- 若现有文件已使用中文注释或文案，新增内容保持一致。

## 测试规范
- 当前测试以 `backend/tests/` 下的脚本式集成测试为主（非完整 pytest 套件）。
- 运行测试前需准备本地依赖环境（MySQL、Redis 及后端依赖）。
- 目前无强制覆盖率门槛；每次行为变更至少补充或更新一条对应测试路径。

## 提交与 Pull Request 规范
- 提交信息使用 Conventional Commits：`feat:`、`fix:`、`refactor:`、`docs:`，风格与现有简洁中文摘要保持一致。
- 每次提交只聚焦一个逻辑改动。
- PR 需包含：变更摘要、影响路径、验证命令、关联需求/Issue；前端界面变更需附截图。

## 安全与配置注意事项
- 不要提交密钥或凭据，统一使用 `backend/.env`。
- 非本地环境必须替换默认 `SECRET_KEY`。
- `backend/ml_models/` 视为模型产物目录，仅在模型/数据变化时更新。
- 后端关键环境变量：`DATABASE_URL`、`REDIS_URL`、`SECRET_KEY`、`KAFKA_BOOTSTRAP_SERVERS`、`BILIBILI_COOKIE`。
- 严禁提交真实 Bilibili Cookie 或访问令牌。

## 接口文档入口
- Bilibili 相关接口与查阅链接统一维护在 `backend/docs/bilibili_api_reference.md`。
- 新增功能遇到接口用法不确定时，先查该文档，再结合 `http://localhost:8000/docs` 验证本项目实际实现。

## 数据采集与 ETL 说明
- 遵守 Bilibili API 频率限制（约 30 次/分钟），采集间隔建议 `>= 2s`。
- 未配置 `BILIBILI_COOKIE` 时，评论接口可能只能返回少量评论。
- ETL 默认每日 `02:00` 执行；手动运行/回填接口：`POST /api/admin/etl/run-sync`、`POST /api/admin/etl/backfill`。

## 直播集成说明
- 直播弹幕依赖 `bilibili-api-python` + `aiohttp`，WebSocket 场景不要替换为 `httpx`。
- 排查多房间或实时指标问题前，先确认 Kafka/Redis 服务健康。

---
> Source: [wwb-org/bilibili-analyzer](https://github.com/wwb-org/bilibili-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
