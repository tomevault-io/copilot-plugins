## dsh-context-length-window-fix

> 本仓库是**文档型 Agent Skill**（不含可执行代码）。维护守则：

# Repository guidance

本仓库是**文档型 Agent Skill**（不含可执行代码）。维护守则：

- `SKILL.md` frontmatter 的 `name` 必须与目录名一致（`01-context-window`）；`description` 同时说明"做什么 + 何时使用"，不超过 1024 字符。
- 保持渐进披露：`SKILL.md` 只放核心流程与速查，细节放 `references/`。
- 示例一律使用占位符（`my-relay`、`https://relay.example.com`、`MY_RELAY_API_KEY`、`<用户名>`），**禁止出现真实用户名、绝对个人路径、真实中转站地址、真实密钥环境变量名**。
- 字段名或行为变化必须以当前安装的 DSH 源码核实（定位入口见 `references/deep-dive.md`），核实后更新 frontmatter 的 `metadata.baseline`。
- "实测参考值"（各模型上下文窗口）需注明查证日期；过期数据宁可删除不要误导。

---
> Source: [xiaohui5206/DSH-Context-Length-Window-Fix](https://github.com/xiaohui5206/DSH-Context-Length-Window-Fix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
