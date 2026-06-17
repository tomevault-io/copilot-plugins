## command-format

> 在 openclaw-agents 中编写或修改文档时，以下命令须严格遵循结尾格式：

# 命令结尾规范

在 openclaw-agents 中编写或修改文档时，以下命令须严格遵循结尾格式：

- **clawhub install**：须带 `--workdir ~/.openclaw` 且以 `;` 结尾。示例：`clawhub install <slug> --workdir ~/.openclaw;`
- **clawhub uninstall**：须以 `;` 结尾。示例：`clawhub uninstall <slug>;`
- **npx skills remove**：须以 `;` 结尾。示例：`npx skills remove <技能名>;`
- **npx skills add**：须以 `-y -g;` 结尾。示例：`npx skills add <owner/repo> --skill <名> -y -g;`

适用于：CLAWHUB-SKILLS.md、README.md、SKILLS-EVALUATION.md、SKILLS-SH-SKILLS.md、TOOLS.md、config/README 等中的代码块、表格与模板示例。

---
> Source: [partme-ai/teams-of-agents](https://github.com/partme-ai/teams-of-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
