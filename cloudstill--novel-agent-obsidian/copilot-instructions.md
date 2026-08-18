## novel-agent-obsidian

> - 不得直接覆盖 vault/07_chapters/ 中的正式正文。

# 小说项目规则

- vault/ 是小说资料的唯一来源。
- 不得直接覆盖 vault/07_chapters/ 中的正式正文。
- AI 生成内容只能写入 vault/90_generated/。
- 未经用户确认，不得更新 vault/04_timeline/current_state.md 和正式伏笔状态。
- 不得默认读取或发送整个 Vault；上下文必须由场景卡的关联字段确定性构建。
- 每次生成请求必须保存上下文清单、输入文件哈希和提示词哈希。
- 不得读取、显示或提交 .env。
- Obsidian 插件的 API Key 必须使用 Obsidian SecretStorage；插件不得读取项目根目录 .env。
- Obsidian 插件只能读取当前场景及其确定性关联资料，不能默认发送整个 Vault。
- Obsidian 插件生成结果只能写入 vault/90_generated/，不能覆盖正式正文或权威状态。
- 测试不得调用真实付费 API，除非用户明确要求。
- 修改工作流后必须运行离线测试。

---
> Source: [Cloudstill/novel-agent-obsidian](https://github.com/Cloudstill/novel-agent-obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
