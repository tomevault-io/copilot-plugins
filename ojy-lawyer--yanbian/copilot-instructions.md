## yanbian

> 本文件只约定项目内协作方式，不扩大用户授权，也不替代宿主工具、安全策略或用户所在设备的约束。

# 砚边：给协作 Agent 的入口

本文件只约定项目内协作方式，不扩大用户授权，也不替代宿主工具、安全策略或用户所在设备的约束。

- **用户要求在自己的电脑部署**：先完整阅读 [Agent 部署手册](docs/AGENT_DEPLOYMENT.md) 与 [手动部署说明](docs/DEPLOYMENT.md)，再按用户所选 GPT（Codex）/DeepSeek 路线推进。先盘点、复用已有组件，不因读取本仓库而自动安装任何东西。
- **用户只问问题、审阅代码或完善文档**：只做该任务，不顺手安装、登录、启停服务或修改信任设置。
- **开发界面**：实际 Word 侧栏入口是 `apps/word-host/sidecar.html`；遵循 `apps/word-host/DESIGN.md`。默认只读问答，明确修改才经 Word 原生修订写入；DeepSeek 不读取或携带 GPT 来源绑定。
- **本机数据**：保留既有设置、文档历史和登录态。不得把真实凭据、私钥、个人配置、业务材料或其他应用的认证文件放进版本控制或公开日志。
- **验证**：离线测试、模拟宿主界面、Word 实际加载、真实模型问答和真实修订分别报告。缺少证书、权限或本机界面能力时明确阻塞，不把替代测试当成功。
- **发布**：仅在用户要求时发布。通过 `scripts/public-source-files.json` 和 `scripts/export-public-source.ps1` 导出并核对清单；不得直接公开开发目录或其旧历史。保留 LICENSE、NOTICE 及署名。

普通代码测试入口为 `node scripts/test-node.js`；可选界面测试为 `node scripts/test-node.js --ui`，依赖已有浏览器及 Puppeteer Core。不要为了部署运行功能而默认安装可选开发测试依赖。

---
> Source: [OJY-lawyer/yanbian](https://github.com/OJY-lawyer/yanbian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
