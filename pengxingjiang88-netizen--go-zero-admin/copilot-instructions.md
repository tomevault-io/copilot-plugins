## go-zero-admin

> - [AI 技能体系使用指南](docs/AI技能体系使用指南.md)

# GZ Admin AI 开发规范入口

## 技能体系总入口

- [AI 技能体系使用指南](docs/AI技能体系使用指南.md)

## 对话语言

在本仓库内工作时，AI 必须使用简体中文与用户沟通。

## 强制技能加载

本项目的项目级技能位于：

```text
skills/gz-admin-framework/SKILL.md
```

推荐在本机终端执行一次：

```bash
make install-codex-skills
```

该命令会把项目技能镜像到 `.codex/skills/gz-admin-framework`，并安装 Codex hooks：

- `SessionStart`：会话启动时加载项目技能上下文。
- `UserPromptSubmit`：用户提交开发请求时，自动注入“必须先加载技能”的流程约束。
- `PreToolUse`：执行编辑或命令前，拦截危险命令和典型 SQL 不安全拼接。
- `Stop`：会话结束时做收尾清理。

当任务涉及以下任一内容时，AI 必须先读取该技能，再执行搜索、编辑、生成代码或测试命令：

- go-zero 后端、API、RPC、model、middleware、handler、logic
- Ant Design Pro Vue3 TypeScript 前端
- SQL、迁移、数据库模型、SQL 注入防护
- RBAC、菜单、角色、用户、权限编码
- 日志、审计、traceId、请求日志、错误日志
- Swagger、上传、定时任务、Redis、缓存、安全配置

执行顺序必须为：

1. 读取 `skills/gz-admin-framework/SKILL.md`
2. 根据任务类型读取其 `references/` 下对应规范
3. 实施代码修改
4. 执行必要检查，至少包含后端变更的 `make check-sql` 和 `go test ./...`

## 阻断规则

- 未读取技能不得直接改代码。
- 涉及 SQL 的业务值必须使用 `?` 参数占位和 `args` 绑定。
- 表名、字段名、排序字段不能来自请求参数，只能来自固定常量或白名单。
- API 层不得绕过 RPC 直接实现核心业务，除非用户明确批准。
- 后台接口必须有权限编码，公开接口必须说明原因。
- 新增或修改注释必须使用中文。
- 代码完成前必须说明已执行的检查命令。

---
> Source: [pengxingjiang88-netizen/go-zero-admin](https://github.com/pengxingjiang88-netizen/go-zero-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
