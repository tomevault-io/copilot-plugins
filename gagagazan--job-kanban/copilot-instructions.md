## job-kanban

> 这个仓库是一个面向公开开源维护的 Next.js 静态 Web 应用。后续 coding agent 修改代码时，请优先遵守本文件和仓库现有实现。

# Agent Guidelines

这个仓库是一个面向公开开源维护的 Next.js 静态 Web 应用。后续 coding agent 修改代码时，请优先遵守本文件和仓库现有实现。

## 沟通与代码风格

- 与维护者沟通时使用中文。
- 编写代码时尊重当前库中的规范、已有测试和既定技术选型；不要凭训练记忆假设 Next.js 行为。
- 不确定实现细节时，先通过仓库文件、测试、官方文档或现有代码确认，再修改。
- 保持改动聚焦，避免顺手重构无关 UI、数据结构或文档。

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## 项目约束

- 应用是 local-first 工具，不引入后端服务、账号体系或外部数据库。
- 用户数据只应保存在浏览器 `localStorage`，除非维护者明确要求改变产品范围。
- 应用配置为 Next.js 静态导出，`npm run build` 输出 `out/`，部署目标是 Cloudflare Pages 或其他静态站点服务。
- UI 文案以中文为主，可以保留必要英文术语，如 Prompt、Provider、Kanban、localStorage。
- 不要添加会把岗位数据、JD、备注、AI 会话链接上传到第三方服务的逻辑。

## 数据模型

本地存储拆成四份：

- `job-kanban.cards`
- `job-kanban.stages`
- `job-kanban.providers`
- `job-kanban.preferences`

导入 / 导出使用带版本号的 JSON bundle。当前导入策略是整包替换，不做 merge。修改数据结构时必须同步：

- `lib/types.ts`
- `lib/schema.ts`
- `lib/storage.ts`
- README 中的数据说明
- 相关测试

## URL 与 Markdown 安全

- 外部链接只允许 `http:`、`https:`、`mailto:`。
- AI Provider URL、阶段 URL Override、会话链接、Markdown 导出链接都必须经过安全协议过滤。
- 不要为了兼容 deeplink 放开 `javascript:`、`data:` 等可执行或高风险协议。
- 修改模板渲染或 URL 编码时，补充包含 `#`、`&`、`=`、中文和空格的回归测试。

## 验证命令

常规改动完成后至少运行：

```bash
npm run lint
npm run test:run
npm run build
npm audit --omit=dev
```

只修改文档时可以只运行：

```bash
npm run lint
```

如果修改依赖、脚本、CI 或 Next.js 配置，必须运行完整验证命令。

---
> Source: [gagagazan/job_kanban](https://github.com/gagagazan/job_kanban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
