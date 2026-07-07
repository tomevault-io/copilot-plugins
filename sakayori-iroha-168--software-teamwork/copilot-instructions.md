## software-teamwork

> <!-- TRELLIS:START -->

<!-- TRELLIS:START -->
# Trellis Instructions

These instructions are for AI assistants working in this project.

This project is managed by Trellis. The working knowledge you need lives under `.trellis/`:

- `.trellis/workflow.md` — development phases, when to create tasks, skill routing
- `.trellis/spec/` — package- and layer-scoped coding guidelines (read before writing code in a given layer)
- `.trellis/workspace/` — per-developer journals and session traces
- `.trellis/tasks/` — active and archived tasks (PRDs, research, jsonl context)

If a Trellis command is available on your platform (e.g. `/trellis:finish-work`, `/trellis:continue`), prefer it over manual steps. Not every platform exposes every command.

If you're using Codex or another agent-capable tool, additional project-scoped helpers may live in:
- `.agents/skills/` — reusable Trellis skills
- `.codex/agents/` — optional custom subagents

Managed by Trellis. Edits outside this block are preserved; edits inside may be overwritten by a future `trellis update`.

<!-- TRELLIS:END -->

## 项目前端说明

- 前端源码放在 `apps/web/src/`。
- 分支、PR、提交和合并策略以 `CONTRIBUTING.md` 为准；当前前端工作默认通过个人 fork 向主仓库 `develop` 发起 PR。
- 面向团队成员的协作流程见 `docs/collaboration/frontend-workflow.md`。
- 面向 agent 的前端规范见 `.trellis/spec/frontend/index.md`。
- 涉及前端开发、分支、PR、Lint 或 CI 时，优先加载项目级 `frontend-workflow` skill。

## Docker 与本地启动说明

- 本地联调入口见 `README.md`、`deploy/README.md` 和 `docs/runbooks/local-integration.md`。
- 默认 host-run 本地完整联调命令是：
  `cp .env.example .env.local`，
  `./scripts/local/start.sh`，
  `cd apps/web && bun install && bun run dev`。
- 云端依赖 Docker app stack 启动命令是：
  `cp deploy/docker/cloud.env.example .env.docker.cloud`，
  填写云端 PostgreSQL/Redis/对象存储/Knowledge runtime/PaddleOCR/model provider 后运行
  `./scripts/docker/start.sh`。
- `./scripts/local/start.sh` 是唯一标准 host-run 本地 setup/start 入口：先检查 `.env.local`、Docker、Go、Python、uv、psql、curl 等宿主机环境和版本，再按需准备本地 Go tools、`goose@v3.27.0`、服务二进制、Docker infra images、Knowledge runtime `.venv`/artifact，最后启动 infra、migration、seed、runtime 和后端服务。
- `start.sh` 不能只因为 `.local/tools/*` 或 `.local/bin/*` 存在就复用；本地 Go 产物必须通过源码 fingerprint/stamp 或等价机制确认新鲜。默认启动应重建过期产物，`--skip-prepare` 遇到过期产物应失败并提示重新运行不带 `--skip-prepare`。Knowledge runtime `.venv` 必须校验 runtime profile 和 `pyproject.toml` / `uv.lock` / `download_deps.py` 依赖输入变化。
- `./scripts/local/stop.sh` 只停止 `start.sh` 记录在 `.local/run/` 里的 host-run 后端和 Knowledge runtime 进程组；不会停止 Docker infra，也不会停止前端 Vite dev server。
- `./scripts/local/clean.sh` 先执行 stop，再删除本地 infra Compose 容器和 PostgreSQL/MinIO/Elasticsearch 数据卷；不会删除 Docker images、`.env.local`、`.local/tools` 或 `.local/bin`。非交互确认用 `--yes`。
- 根级 Docker Compose `deploy/docker-compose.yml` 只允许拉取并启动基础设施：`postgres`、`redis`、`minio`、`minio-init`、`elasticsearch`。默认 host-run 联调中 Auth、File、Knowledge、QA、Document、AI Gateway、Gateway、Parser 和前端都必须按文档在宿主机启动。
- `deploy/docker-compose.cloud.yml` 是独立 cloud Docker app stack，可以 `build:` 业务服务和前端镜像，但必须外接云端 PostgreSQL、Redis、对象存储、Knowledge runtime、PaddleOCR/OCR 和模型 provider；不能启动本地 OCR、Knowledge runtime worker、Elasticsearch、PostgreSQL、Redis 或 MinIO。
- Cloud Docker app stack 是明确批准的第二条启动路径，不是根级本地 Compose 的例外扩散；模板默认 `DOCKER_SEED_ENABLED=false`，不要把 `local-dev-*`、`local-demo-*`、`change-me` 或 `<...>` 占位 secret 用于 cloud seed 或网络可达环境。
- `config/` 是唯一默认配置来源；根 `.env.example` 是本地 secret 模板，用户复制成未跟踪的 `.env.local`。启动脚本只检查 `.env.local` 是否存在，不创建或覆盖它，并通过 `config/ctl` 渲染 `.local/config/<profile>.env` 和 `.env.sh`。
- 当前源策略是默认官方源、国内网络显式 `--china`；不要把缺少 active DaoCloud/TUNA/goproxy 默认值标记为回归。
- Go 本地基线是 Go `1.25.x`（脚本最低接受 `go1.25.1`，当前本地基线可用 `go1.25.4`）；迁移工具固定 `goose@v3.27.0`，不要未经说明改回需要更高 Go patch 的版本。
- `start.sh` 必须在构建 `config-ctl`、安装 goose 和构建 seed helper 前读取 `.env.local` 中的 `GOPROXY`、`GOSUMDB`、`GOPRIVATE`、`GONOPROXY`、`GONOSUMDB`、`GOINSECURE`，这样企业 Go 源可以影响首次准备。`--china` 只影响本次运行，不改写 `.env.local`。
- `UV_DEFAULT_INDEX` 属于宿主机 uv/Python 包索引配置，默认放在 `config/base.yaml` 且默认指向官方 PyPI；不要把 uv 慢误判成 Docker registry 问题。中国大陆网络用显式 `--china` 或本机私有覆盖，不要把默认配置改成第三方镜像。
- Docker 镜像源、registry rewrite、daemon mirror、proxy、pull 卡顿和 WSL 内存排障见 `docs/runbooks/docker-image-pull-environment.md`。默认 host-run 路径使用官方 pinned infra images；面向中国大陆网络运行 `./scripts/local/start.sh --china` 显式启用 registry rewrite。Cloud Docker app stack 的基础镜像、Go module 源和 Alpine apk 源通过 `.env.docker.cloud` 中的 `DOCKER_IMAGE_REGISTRY_PREFIX`、`GO_DOCKER_GOPROXY`、`GO_DOCKER_GOSUMDB`、`ALPINE_MIRROR` 等覆盖；优先级为 `registry rewrite > daemon mirror > proxy`。
- GitHub release/raw 下载慢时，不要改写 committed `pyproject.toml`、`uv.lock` 或 OpenAPI 契约。Knowledge runtime 依赖和 artifact 由 `./scripts/local/start.sh` 按当前源模式准备；中国大陆网络使用 `--china`。
- 改 Compose、基础设施镜像 tag、cloud Docker app stack、镜像源、Docker 环境诊断、Docker 文档或相关 Trellis spec 时，必须运行 `python3 scripts/check_docker_policy.py`、相关单元测试、`CONFIG_SECRET_FILE=.env.example ./scripts/config/load-profile.sh --print-compose-env`、`docker compose -f deploy/docker-compose.yml --env-file .local/config/dev.env config --quiet` 和 `docker compose -f deploy/docker-compose.cloud.yml --env-file deploy/docker/cloud.env.example config --quiet`。
- 不要把正常路径改成 `latest` 镜像；遇到镜像源异常时先按 Docker runbook 排查并记录环境阻断。

## 本地私有说明

- Agent 可在存在时读取 `.agents/local/AGENTS.local.md` 作为本机私有补充说明。
- 该文件应保持 Git 忽略状态，用于记录机器特定的代理、账号或凭据相关要求。

---
> Source: [Sakayori-Iroha-168/Software_Teamwork](https://github.com/Sakayori-Iroha-168/Software_Teamwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
