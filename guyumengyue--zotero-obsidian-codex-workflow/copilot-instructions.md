## zotero-obsidian-codex-workflow

> [你负责在仓库根目录帮助用户初始化和维护这套 Zotero-Obsidian-Codex 文献工作流模板。]

# 角色

[你负责在仓库根目录帮助用户初始化和维护这套 Zotero-Obsidian-Codex 文献工作流模板。]

---

# 工作边界

- 仓库根目录是 Obsidian vault 根目录。
- `Research vault/` 是日常文献处理工作区，内部目录名固定，不要重命名。
- 优先使用统一入口 skill：`$zotero-obsidian-codex-workflow`。
- 当用户要做 clone 后初始化、迁移、分享、校验、轻量微调时，使用 `$zotero-obsidian-codex-setup` Skill。
- 当用户要读文献、补笔记、写综述、设计实验时，不要停留在仓库根目录；切换到 `Research vault/`，遵守其中的 `AGENTS.md` 和 `$literature-processor`。
- 如果当前位于仓库根目录，且 `.obsidian/plugins/obsidian-zotero-desktop-connector/data.json` 不存在，优先执行 repo 内的 setup 检查与初始化流程，再继续其他工作。

---

# 初始化目标

1. 检查或指导安装：
   - `Obsidian`
   - `Zotero`
   - `Better BibTeX for Zotero`
   - Obsidian 社区插件 `Dataview`
   - Obsidian 社区插件 `Zotero Integration`
2. 通过仓库内 setup 资源写入：
   - `.obsidian/plugins/obsidian-zotero-desktop-connector/data.json`
3. 保持默认导出目标：
   - `Research vault/references.bib/library.bib`
4. 初始化完成后，把后续工作交接到：
   - `Research vault/`

---

# 公开模板护栏

- 这是公开模板仓库，不要把真实 PDF、真实文献笔记、私有 BibTeX 导出、缓存、workspace 状态或测试残留加入版本控制，除非用户明确要做私有本地修改。
- 不要把 Obsidian 社区插件二进制打包进仓库。
- 对外路径说明一律用相对路径，不写本机绝对路径。

---
> Source: [guyumengyue/zotero-obsidian-codex-workflow](https://github.com/guyumengyue/zotero-obsidian-codex-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
