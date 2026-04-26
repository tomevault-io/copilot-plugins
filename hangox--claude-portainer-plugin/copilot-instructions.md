## claude-portainer-plugin

> 这是一个 Claude Code 插件，通过 Portainer REST API 管理 Docker 容器和服务。插件以 `.claude-plugin` 标准格式发布，支持一键安装。

# CLAUDE.md

## 项目概述

这是一个 Claude Code 插件，通过 Portainer REST API 管理 Docker 容器和服务。插件以 `.claude-plugin` 标准格式发布，支持一键安装。

## 项目结构

```
claude-portainer-plugin/
├── .claude-plugin/plugin.json              # 插件元数据
├── skills/portainer/
│   ├── SKILL.md                            # 核心 Skill（≤500 行）
│   └── references/
│       ├── api-reference.md                # Portainer 原生 API 参考
│       ├── docker-proxy-api.md             # Docker 代理 API 参考
│       ├── workflow-examples.md            # 工作流示例和最佳实践
│       └── acceptance-checklist.md         # 对话场景验收清单
├── tests/                                  # 自动化测试（Portainer CE 2.24.1 + DinD）
├── README.md
└── LICENSE (MIT)
```

## 开发规范

- SKILL.md 必须保持在 500 行以内，详细内容放到 references/
- 所有环境变量统一使用 `PORTAINER_*` 前缀
- curl 示例中使用 `$PORTAINER_URL`、`$PORTAINER_API_KEY`、`$PORTAINER_ENDPOINT_ID`、`$AUTH`、`$EP` 变量
- 不允许硬编码任何服务器地址、密码或个人配置
- 版本兼容性：最低 Portainer CE 2.19+，已验证 2.24.1 和 2.33.7

## 测试

```bash
cd tests && npm install && npx tsx run-tests.ts
```

测试会自动启动 Portainer + Docker 环境，执行 28 个测试用例，完成后自动清理。需要本机安装 Docker。

## 已知的 API 版本差异

- Stack 创建路径：2.33+ 用 `/api/stacks/create/2/string`，2.24.x 用 `/api/stacks/create/standalone/string`
- 系统状态：2.33+ 用 `/api/system/status`，旧版用 `/api/status`
- Docker 代理创建操作返回 200（非标准 Docker API 的 201）
- 镜像拉取返回流式 NDJSON
- API Key 创建需要 body 中包含 `password` 字段
- Endpoint 注册需要 `multipart/form-data`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hangox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
