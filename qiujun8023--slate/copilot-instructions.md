## slate

> 本文件给本地 AI 代理使用，说明 Slate 仓库内必须遵守的协作和发布规则。面向人的贡献说明见 [CONTRIBUTING.md](CONTRIBUTING.md)。

# Agent Guide

本文件给本地 AI 代理使用，说明 Slate 仓库内必须遵守的协作和发布规则。面向人的贡献说明见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 基本规则

- 默认在 `master` 上开发，提交风格遵循 Conventional Commits。
- 不要回滚用户已有改动；如遇到无关的脏工作区，忽略即可。
- 修改前先阅读相关模块 README：后端看 `backend/README.md`，前端看 `frontend/README.md`，共享 schema 看 `shared/README.md`，固件看 `firmware/README.md`。
- 前端 UI 改动必须遵守 `frontend/README.md` 的 Mono Press 设计系统。
- 固件改动影响 partition、NVS、同步协议或 OTA 路径时，必须在最终说明里明确风险和验证结果。

## 提交规则

- 提交信息遵循仓库历史：`<type>(<scope>): <中文摘要>`，body 用 `- ` 条目说明具体改动。
- 需要提交 body 时，使用单个 `-m` 传入完整提交信息，例如 `git commit -m $'subject\n\n- item'`。
- 不要在提交信息里加入任何 AI 署名、生成标识、协作者 trailer 或工具标记。

## 常用校验

发布前或大改后优先执行：

```bash
bun run format:check
bun run lint
bun run typecheck
bun run --cwd backend test
bun run --cwd frontend build
```

固件改动还需要 ESP-IDF 5.5.x 构建：

```bash
source $IDF_PATH/export.sh
idf.py -C firmware build
```

## 发布规则

Slate 使用单一产品版本号。一个 `vX.Y.Z` tag 同时发布：

- GHCR Docker 镜像：backend + frontend dist
- ESP32-S3 固件：完整烧录包和 OTA 包
- GitHub Release：release notes、固件附件、校验和、部署用 `compose.yml` / env 示例

不要为 backend 和 firmware 分别创建 release。

### 发版前必须同步版本号

以下位置必须与 tag 去掉 `v` 后一致：

- `package.json`
- `backend/package.json`
- `frontend/package.json`
- `shared/package.json`
- `bun.lock` 中 workspace package 的版本记录
- `firmware/sdkconfig.defaults` 中的 `CONFIG_APP_PROJECT_VER`

例如发布 `v0.2.0` 时，上述版本都必须是 `0.2.0`。

更新 package 版本后运行一次 `bun install`，让 `bun.lock` 同步 workspace 版本。不要只改 `package.json`。

### 只能使用 annotated tag

Release notes 来自 tag body。不要创建 lightweight tag，不要手动编造 GitHub Release 内容。

推荐命令：

```bash
git tag -a v0.2.0
git push origin v0.2.0
```

tag message 示例：

```text
Slate v0.2.0

- 后端 / Web：新增 ...
- 固件：修复 ...
- 兼容性：推荐固件 v0.2.0
```

### CI 发布流程

推送 `vX.Y.Z` tag 后，`.github/workflows/release.yml` 会自动：

1. 校验 tag 是 annotated tag，且 tag body 非空
2. 校验各模块版本号与 tag 一致
3. 跑格式、lint、typecheck、后端测试、前端构建
4. 构建并推送 Docker 镜像 tag：`vX.Y.Z`、`X.Y`、`latest`
5. 构建固件并生成带版本号的 `.bin`
6. 创建或更新 GitHub Release
7. 上传固件 `.bin`、sha256、部署用 `compose.yml` / env 示例到 Release assets

`docker.yml` 和 `firmware.yml` 是 `master` 的滚动构建，不代表稳定版本。正式版本只看 GitHub Releases 和 `vX.Y.Z` tag。

### 禁止事项

- 不要手动编辑 GitHub Release notes；应修改 annotated tag body 后重新推送 tag。
- 不要在 release workflow 之外推送 `vX.Y.Z` 镜像 tag。
- 不要让 package 版本、固件版本和 Git tag 不一致。
- 不要把正式 release 建在非 `vX.Y.Z` tag 上。
- 不要重跑旧版本 tag；`release.yml` 会拒绝非最高 `vX.Y.Z` tag，避免 Docker `latest` 回滚。

---
> Source: [qiujun8023/slate](https://github.com/qiujun8023/slate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
