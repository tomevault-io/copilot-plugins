## minutes

> 会议录音转文字后端服务，提供异步 Job API、SSE 事件流和 OpenAI 兼容同步接口。

# minutes 项目指南

会议录音转文字后端服务，提供异步 Job API、SSE 事件流和 OpenAI 兼容同步接口。
推理通过 HTTP 调用外部 OpenAI-compatible STT 服务（如 funasr-server 或 Speaches）。

## Call Flow Cheat Sheet

- Gateway entry: `src/minutes_gateway/app.py`
- Job API: `src/minutes_gateway/routers/jobs.py`
- OpenAI-compatible API: `src/minutes_gateway/routers/openai.py`
- Queue entry: `src/minutes_orchestrator/actors.py` -> `src/minutes_orchestrator/services.py`
- Inference entry: `src/minutes_inference/actors.py` -> `src/minutes_inference/service.py` -> `src/minutes_inference/engines/`
  - `remote_stt.py`: HTTP client，调用外部 STT 服务 (`POST /v1/audio/transcriptions`)
  - `adapter.py`: 将 OpenAI verbose_json 响应转为 `TranscriptDocument`
  - `fake.py`: 测试用 Fake engine
- Retry exhausted: `src/minutes_orchestrator/actors.py::handle_orchestrator_retry_exhausted`, `src/minutes_inference/actors.py::handle_inference_retry_exhausted`
- Shared core: `src/minutes_core/`
- Local smoke: `scripts/local_run_job.py`
- Docker entry: `docker-compose.yml` + `docker/*.Dockerfile`

## 工具链（禁止替换）

| 层级 | 包管理器 | 测试框架 | Linter/Formatter | 配置文件 |
|------|---------|---------|-----------------|---------|
| 后端 (Python) | **uv** | **pytest** | **ruff** | `pyproject.toml` + `uv.lock` |

常用命令：
- 安装依赖: `make install`
- 数据库迁移: `make db-upgrade`
- 格式化: `make format`
- 检查: `make lint`
- 全量检查: `make check`
- 环境自检: `make check-env`
- 测试: `make test`

## 架构

Pipeline:
`upload -> create job -> prepare_job -> transcribe_job -> finalize_job -> transcript/export`

- Gateway: FastAPI，对外提供 REST、SSE 和 OpenAI 兼容接口
- Orchestrator: Dramatiq worker，负责 `ffprobe/ffmpeg`、状态机和收尾
- Inference: Dramatiq worker，通过 HTTP 调用外部 STT 服务（RemoteSTTEngine）
- Core: 配置、数据库、仓储、事件总线、导出与存储

## STT 后端配置

minutes 本身不包含 ASR 模型，而是作为任意 OpenAI-compatible STT API 的纯消费者。

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `MINUTES_STT_BASE_URL` | `http://localhost:8101` | STT 服务地址 |
| `MINUTES_STT_API_KEY` | (空) | STT 服务 API key |
| `MINUTES_STT_TIMEOUT_SECONDS` | `600` | 单次转写超时（10分钟） |

可用的 STT 后端:
- **funasr-server** (`~/workspaces/tool/funasr-server/`): FunASR 引擎，默认 8101 端口
- **Speaches** (`~/workspaces/tool/speaches/`): Whisper 引擎，默认 8103 端口

## 开发服务管理

需要服务时，先检查再启动，尽量不要要求用户手工开多个终端。

| 服务 | 检查 | 启动 | 何时需要 |
|------|------|------|---------|
| Redis | `redis-cli ping` | `make redis` | 队列与事件流 |
| Gateway | `curl -sf http://127.0.0.1:8000/health` | `make dev-gateway` | API 开发 |
| Orchestrator | `pgrep -f 'dramatiq minutes_orchestrator.actors'` | `make dev-orchestrator` | 预处理调试 |
| Inference | `pgrep -f 'dramatiq minutes_inference.actors'` | `make dev-inference` | 推理调试 |
| STT (FunASR) | `curl -sf http://127.0.0.1:8101/health` | 在 funasr-server 项目: `make dev` | 真实转写 |
| STT (Speaches) | `curl -sf http://127.0.0.1:8103/health` | 独立服务 | Whisper 转写 |

轻量模式：
- 仅跑单机 smoke: 直接用 `scripts/local_run_job.py`
- 调 API 但不测异步 worker: 启动 Gateway 即可
- 调异步链路: Gateway + Redis + 对应 worker

## 音频 smoke

- Windows 测试目录 `C:\temp\meetings` 在 WSL 中对应 `/mnt/c/temp/meetings`
- Fake smoke:
  - `make smoke-fake AUDIO=/mnt/c/temp/meetings/demo.wav`
- Real smoke (需要 STT 服务运行):
  - `make smoke-real AUDIO=/mnt/c/temp/meetings/demo.wav`
  - 可通过 `--stt-base-url` 参数指定 STT 服务地址

## 测试基础设施

- 共享 Fake / Builder / 断言: `tests/helpers.py`（`RecordingQueue`, `RecordingEventBus`, `build_transcript_document`, `extract_job_id`, `error_message`, `assert_reasonable_error`）
- Orchestrator/Inference 服务环境: `tests/orchestrator/conftest.py`（`ServiceEnv` + `create_test_job` + `service_env` fixture）
  - `ServiceEnv.make_orchestrator()` / `make_inference()` / `get_job()` 减少样板代码
  - Inference 测试通过 `tests/inference/conftest.py` re-export `service_env` fixture
- Gateway 测试环境: `tests/gateway/conftest.py`（`GatewayEnv` + `GatewayHarness` + `gateway_harness_factory` + `async_gateway_env`）
- SSE 异步测试: `tests/gateway/test_sse_async.py`（使用 `httpx.AsyncClient` + `ASGITransport`）
- RemoteSTT 测试: `tests/inference/test_remote_stt.py` + `test_remote_stt_branches.py`（HTTP 调用、错误映射）
- Adapter 测试: `tests/inference/test_adapter.py`（verbose_json → TranscriptDocument 转换）
- Service 分支测试: `tests/inference/test_service_branches.py`（NOOP 状态、重试、事件发布）

## 代码规范

- 注释、文档字符串: 简体中文
- 日志、代码内文本: 英文
- 显式优于隐式，关键调用优先显式参数
- 不要在 repository 内部隐式提交事务；由调用方决定 `commit/rollback`
- 新增路由、服务入口、状态流转或模型配置时，必须同步更新本文件和对应文档

## 提交前自检（必须执行）

- 每次准备提交前，先运行 `make check`
- 然后启动一个 subagent 对本次变更做自检，至少覆盖：
  1. `CLAUDE.md` 是否需要同步
  2. 新增/修改的状态机、actor、router、脚本是否有测试覆盖
  3. 新增日志是否为英文，新增用户可见文本是否为简体中文
  4. 文档命令是否仍与仓库实际工具链一致
  5. 是否引入未锁定依赖或未记录的运行前提
- subagent 发现问题时，先修复，再提交

## 文档规范

- 计划文档放 `docs/plans/`
- 评审文档放 `docs/reviews/`
- 指南文档放 `docs/guides/`
- 本地测试说明与模型准备步骤变更时，务必同步更新 `README.md` 和 `docs/local-testing.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/code404c) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
