## nest-to-server-less

> 贡献代码时请注意：这是一个通过 AWS SAM 部署的 NestJS 项目，推荐使用 pnpm 完成所有工作流，确保基础设施产物与现有流水线保持一致。

# 仓库指南

贡献代码时请注意：这是一个通过 AWS SAM 部署的 NestJS 项目，推荐使用 pnpm 完成所有工作流，确保基础设施产物与现有流水线保持一致。

## 项目结构与模块组织

- `src/` 存放 Nest 模块、控制器和服务；`src/form-data`、`src/github` 等功能文件夹封装对应领域逻辑。
- `src/lambda.ts` 负责将 Nest 应用适配到 Lambda，`src/main.ts` 仍然是本地启动入口。
- `prisma/` 包含 schema、迁移脚本与种子数据工具，这些会被 Lambda Layer 引用。
- `test/` 保存 e2e 用例（`app.e2e-spec.ts`）；单元测试请与源码放在一起，命名为 `*.spec.ts`。
- `template.yaml`、`samconfig.toml` 与 `deploy.sh` 描述 SAM 部署栈；`layer/` 通过 `build:lambda` 构建，勿手动加入版本控制。

## 构建、测试与开发命令

- `pnpm start:dev`：本地热重载运行 Nest 服务器。
- `pnpm build`：将 TypeScript 编译到 `dist/`，用于传统托管方式。
- `pnpm build:lambda`：打包 Lambda 代码与运行时 Layer。
- `pnpm sam:local`：本地模拟 API Gateway，可搭配 `pnpm sam:logs` 查看远端日志。
- `pnpm sam:build` + `pnpm sam:deploy`：执行 SAM 打包与部署流程；`pnpm deploy:aws` 封装了脚本化部署。

## 代码风格与命名约定

默认使用两空格缩进的 TypeScript，导入按作用域排序（先 Nest，再三方，再本地模块）。类、模块与 DTO 使用 `PascalCase`，Provider 与函数使用 `camelCase`，目录保持 `kebab-case`。提交前务必运行 `pnpm lint` 与 `pnpm format`；避免提交 `prisma/` 之外的 Prisma 生成产物。

## 测试指南

使用 Jest（ts-jest）。单元测试命名为 `*.spec.ts` 并置于源码旁；e2e 测试位于 `test/`，命名为 `*.e2e-spec.ts`。常规提交运行 `pnpm test`，重要合并前运行 `pnpm test:cov`，部署前建议执行 `pnpm test:e2e` 验证 Lambda 接口。外部服务优先做 mock，但 Supabase/Prisma 流程在 e2e 中尽量走真实调用。

## 提交与 PR 规范

遵循 Conventional Commits（如 `feat:`、`fix:`、`chore:`）。PR 描述需概述改动范围，标注对 `template.yaml` 或环境文件的变更，并关联跟踪 issue。附上 `pnpm lint` 与相关测试结果；若修改了基础设施导致运行时行为变更，提供部署截图或说明。

## 无服务器部署提醒

`.env.production` 基于 `.env.production.template` 生成，禁止提交真实密钥。修改 Prisma schema 或基础设施资源后，请重新执行 `pnpm build:lambda` 并使用 `sam validate` 校验 `template.yaml`。

---
> Source: [iceberg211/nest-to-server-less](https://github.com/iceberg211/nest-to-server-less) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
