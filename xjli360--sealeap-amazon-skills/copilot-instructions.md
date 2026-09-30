## sealeap-amazon-skills

> - 本目录只代表一个来源平台的独立工作流集合；平台代号为 `athena`，新增平台先检查 `app` 中所有目录名，不能复用已占用名称。

# Athena 集合维护与使用

- 本目录只代表一个来源平台的独立工作流集合；平台代号为 `athena`，新增平台先检查 `app` 中所有目录名，不能复用已占用名称。
- 用户寻找本集合技能时优先使用 `sealeap-athena-find-skills` 的本地检索。已明确的业务任务直接使用对应 Skill，不强制绕到搜索或安装流程。
- 每个包使用同名目录和 `sealeap-athena-...` frontmatter 名称，包含 `SKILL.md`、`agents/openai.yaml`、`references/playbook.md` 与 `references/tool-routing.md`。
- 修改技能时同步本目录 `catalog.json` 和 `README.md`；检查名称唯一、相对引用、工具名与真实能力。只核验结构不能宣称在线业务已通过。
- 技能正文、脚本、元数据和文件名保持中性；来源原件、身份映射、原网关和凭证不能进入本目录。实际业务数据仍须记录真实数据来源、时间和范围。
- 只使用当前可用的替代工具、官方账户连接或用户数据；没有对应能力时写明差异和缺口，不制造可调用的接口。
- 更新时在私有暂存区下载、检查并迁移所需变化，不把原始包直接覆盖进本目录。

- 当前为 39 个入口，原 226 项业务能力保存在各包 `references/capabilities/`；旧名称映射见 `migration.json`。
- 修改细分能力时同步本包 `references/capabilities.json`、根 `catalog.json`、`migration.json` 和相应 Markdown 索引；每个原能力只对应一个入口。
- 选择模式后只读其指南和必要路由，不加载全部子能力。旧名称由本地搜索解析，不宣称宿主自动支持旧 `$skill-name`。

---
> Source: [xjli360/sealeap-amazon-skills](https://github.com/xjli360/sealeap-amazon-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
