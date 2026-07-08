## memory-gallery

> 本文档是 AI Agent、自动化脚本和后续维护者接手本仓库时的最高优先级项目约束。

# Agent 协作规范

本文档是 AI Agent、自动化脚本和后续维护者接手本仓库时的最高优先级项目约束。

## 项目定位

这是一个轻量回忆相册应用，不是大型多租户 SaaS。功能设计优先保证：

- 亲友成员使用简单。
- 图片和元数据不容易丢。
- 小规模部署和迁移成本低。
- 前后端边界清楚，避免过度工程化。

## 工作原则

- 先阅读 `README.md`、`docs/project-overview.md`、`docs/architecture.md` 和目标目录下的 `README.md`。
- 修改前先确认当前 Git 状态，不能回滚用户已有改动。
- 文档、代码、数据样例要保持一致；新增接口或字段时必须同步更新 `docs/api.md` 和 `docs/data-model.md`。
- 每次完成项目建设、体验调整、功能新增或重要修复后，必须在 `docs/project-log.md` 追加日期记录；涉及设计取舍、存储方式、架构边界或长期维护习惯时，还要同步更新 `memory/decision-log.md`、`docs/progress.md`、`docs/module-map.md` 或相关专题文档。
- 不要把业务代码、运行数据和文档整理混在一个不必要的大改动里。
- 默认使用中文说明，代码标识、命令、接口路径保持原文。

## 禁止事项

- 不要提交 `node_modules/`。
- 不要删除 `uploads/`、`photo-data.json`、`group-data.json`、`story-data.json` 中的真实用户数据，除非用户明确要求。
- 不要硬编码新的生产密码、Token、云平台密钥。
- 不要在未确认数据备份的情况下改动存储格式。
- 不要引入数据库、构建系统或前端框架，除非项目计划明确决定迁移。

## 文档维护要求

每次完成以下变更时，都必须同步更新文档：

- 新增、删除或重命名 API。
- 新增、删除或重命名数据字段。
- 调整目录结构。
- 改变运行、部署、备份或环境变量方式。
- 新增重要功能模块。
- 完成一轮对话式项目建设后，要留下可追溯记录，至少包含完成内容、关键决策、验证结果、风险或后续事项。

## 编码约束

- 前端目前使用原生 ES Modules，优先延续现有模块化方式。
- 后端目前使用 CommonJS，优先延续现有 `require/module.exports` 方式。
- JSON 文件写入应继续使用临时文件 + rename 的原子替换方式。
- 用户可见文本应保持 UTF-8 中文，避免乱码。
- 图片上传、删除、重命名、缩略图生成必须考虑文件系统一致性。

## 验证清单

变更完成前至少检查：

- `npm start` 能启动服务。
- 首页可打开。
- `/api/photos` 可返回 JSON。
- 上传目录和数据文件路径符合当前环境变量。
- 文档链接没有明显失效。

## 相关文档

- [项目总览](./docs/project-overview.md)
- [架构说明](./docs/architecture.md)
- [模块职责](./docs/module-map.md)
- [开发约束](./docs/development-constraints.md)
- [部署与运维](./docs/deployment-operations.md)

---
> Source: [by20999/Memory-gallery](https://github.com/by20999/Memory-gallery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
