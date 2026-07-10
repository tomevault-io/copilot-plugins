## streamserve

> 将运行时配置集中在 `nginx/`（模板位于 `nginx/conf.d/`），可复用的 Shell 工具放在 `scripts/`，基础设施相关工具置于 `infra/`。容器构建文件位于 `docker/`，其中 `Dockerfile` 基于阿里云镜像构建 nginx-rtmp 环境。文档统一存放在 `docs/`，参考素材放入 `assets/`，测试与烟囱检查按目标目录映射至 `tests/`。避免提交二进制文件，优先构建可复现的制品。

# 仓库指南

## 项目结构与模块组织
将运行时配置集中在 `nginx/`（模板位于 `nginx/conf.d/`），可复用的 Shell 工具放在 `scripts/`，基础设施相关工具置于 `infra/`。容器构建文件位于 `docker/`，其中 `Dockerfile` 基于阿里云镜像构建 nginx-rtmp 环境。文档统一存放在 `docs/`，参考素材放入 `assets/`，测试与烟囱检查按目标目录映射至 `tests/`。避免提交二进制文件，优先构建可复现的制品。

## 构建、测试与开发命令
使用 `docker compose up --build streamserve` 构建并启动 nginx-rtmp 栈（镜像由 `docker/Dockerfile` 本地构建，无需访问 Docker Hub）。编辑配置后，通过 `docker compose exec streamserve nginx -t` 校验语法。采用 `./scripts/reload.sh` 在容器内热重载 nginx，使用 `./scripts/package.sh` 导出适用于裸机部署的配置，首次上线云主机时执行 `sudo ./scripts/deploy_streamserve.sh` 进行全自动安装与拉起。

## 编码风格与命名约定
配置与脚本文件命名优先采用小写连字符。环境差异化内容使用 `*.conf.tpl` 模板化。Shell 脚本默认遵循 POSIX sh；若需数组或高级管道控制，可切换 Bash 并启用 `set -euo pipefail`，同时保持两个空格缩进。JSON、YAML 与 Markdown 也保持两个空格缩进与 80 字符软换行。环境变量统一使用大写蛇形命名且语义清晰。

## 测试规范
提交前运行 `docker compose exec streamserve nginx -t`，并执行 `tests/` 中相应的目标化烟囱测试。新增检查应与对应代码同目录维护；Shell 测试文件命名为 `test_<area>.sh`，并将样例数据置于 `tests/fixtures/`。为每个 server block 保持 curl 或 k6 覆盖，金标准输出存放于 `tests/golden/` 以便确定性比对。

## 提交与合并请求规范
遵循 Conventional Commits（如 `feat:`、`fix:`、`docs:`、`ci:`），使用 65 字符以内的现在时摘要。在正文中关联 Issue，概述配置影响并附上测试命令输出。合并请求需给出回滚步骤，对仪表盘或界面改动提供截图，并邀请相关目录维护者评审。

## 安全与配置提示
禁止提交 `.env`、证书或订阅密钥等敏感文件；将其保存在 Git 忽略的 `secrets/` 内，并提供 `.example` 模板。确保各环境的限流与鉴权指令保持启用，对合理例外应记录在 `docs/security.md`。一旦凭据暴露需立即轮换并在提交中说明。

## AI 提示：RTMP 部署作业手册
在请求 AI 助手协助自动化 StreamServe 部署时，请共享以下指令：

> **角色**：以资深 SRE 身份构建名为 StreamServe 的 nginx-rtmp 流媒体服务。
> 
> **背景**：目标环境为 Ubuntu 22.04 云主机，具备 Docker Engine 24+ 与经堡垒机的 SSH 访问。代码仓包含 `nginx/`、`scripts/`、`infra/` 与 `tests/` 目录。
> 
> **目标**：
> 1. 输出 `scripts/deploy_streamserve.sh`，负责安装 Docker、克隆仓库、执行 `docker compose up -d streamserve`、初始化卷，并应用 `nginx/nginx.conf` 及 RTMP 引用文件。
> 2. 输出 `nginx/conf.d/rtmp.conf`，启用 `nginx-rtmp-module`，配置 `listen 1935`、`application live { live on; record off; }`，并将 HLS 代理到 `/hls/`。
> 3. 在 `.env.example` 中列出所需环境变量（`STREAMSERVE_DOMAIN`、`SSL_CERT_PATH`、`PUSH_KEY`、`RTMP_ALLOWED_IPS`），并通过 `docker compose --env-file` 加载。
> 4. 描述对 `rtmp://$host/live/stream` 的推流冒烟测试，并利用 `ffprobe` 或 `ffmpeg -i` 验证播放。
> 
> **实施步骤**：涵盖 `ssh -J bastion user@target` 连接流程、系统补丁与加固、开放 22/80/443/1935 端口、防私有仓库登录及部署后使用 `docker compose ps` 与 `nginx -T` 复核。
> 
> **工具期待**：优先使用 Docker，注明裸机回退方案，提供 `docker compose down --volumes` 回滚操作，并强调使用 `docker compose logs -f streamserve` 收集日志及接入指标监控。

执行前请确认 AI 输出符合上述检查清单。

---
> Source: [sunlulu427/StreamServe](https://github.com/sunlulu427/StreamServe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
