## aioffer-cli

> 本项目面向中国用户。README、操作教程、界面提示、错误说明和贡献指南使用中文；代码标识符、协议字段和第三方专有名称保留原名。

# aioffer-cli 开发约束

本项目面向中国用户。README、操作教程、界面提示、错误说明和贡献指南使用中文；代码标识符、协议字段和第三方专有名称保留原名。

## 不可改变的边界

- 产品名与仓库名为 `aioffer-cli`。
- 执行、配置、简历、投递记录均保存在用户电脑；只监听 `127.0.0.1:19876`。
- 用户自行配置 MinerU 和视觉模型密钥。不得加入 AI Offer 登录、企业云端、收费权益、云数据库、对象存储账号或遥测依赖。
- 扩展后台只连接本地服务。模型与 MinerU 请求由本地服务发出。招聘网站由浏览器正常访问。
- 不得把真实简历、密钥、Cookie、验证码、内部验收记录写入代码、测试夹具、日志或公开提交。
- 没有用户针对真实投递的明确授权，不得用真实招聘岗位做最终提交测试。

## 投递行为保护

原站点驱动与测试位于 `extension/src/control-adapters/`。修改表单观察、填写、回读、ATS 行为前，完整阅读：

1. `docs/upstream/AGENTS.md` 的中文迁移版。
2. `docs/upstream/CONTROL_ADAPTER_ROUTING_POLICY.md`。
3. `docs/upstream/CONTROL_ADAPTER_REAL_PAGE_EVIDENCE.md`。
4. `docs/upstream/ARCHITECTURE.md` 和受影响驱动的现有测试。

这些文件记录上游实现与历史证据，不代表本仓库已通过实站验收。历史用户授权、服务器地址、发布操作和内部系统说明不适用于本仓库。本文件的本地边界及本次用户授权优先。

半自动模式复用同一投递执行器，额外要求提交前的明确确认，且确认绑定当前页面回读哈希。自动模式要求批次最终提交授权。不得用测试用的 `localValidation` 绕过生产租约、中断、停止或提交意图保护。

发布前运行 `pnpm typecheck`、`pnpm test`、`pnpm build`、`pnpm check:cloud`，分别报告自动化覆盖、构建结果和实站业务结果。未实测的更改只能作为草稿，不得宣称所有网站可用。

CLI 产品版本与继承的插件协议版本独立管理。不得通过任意提高最低插件版本绕过兼容性验证。

---
> Source: [Succaiss-applied-ai/aioffer-cli](https://github.com/Succaiss-applied-ai/aioffer-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
