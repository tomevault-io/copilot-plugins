## skillseed

> 这是一个 **Agent Skills 仓库**: 基于 Agent Skills 开放标准, 生成的 skill 兼容 Claude Code、Codex、Cursor 等工具。

# SkillSeed

这是一个 **Agent Skills 仓库**: 基于 Agent Skills 开放标准, 生成的 skill 兼容 Claude Code、Codex、Cursor 等工具。

## 关键约定
- 所有 skill 的 canonical 源在 `skills/<name>/` 下编写与版本管理。
- 用 `/skillseed` 创建新 skill (它会生成到 `skills/<name>/`, 并做合规自检)。
- 用 `scripts/install-skill.sh <name> --tool <claude|codex|cursor> --project|--user` 把 skill 安装到对应工具的可发现路径。

## 开发 skillseed 自身
- 源在 `skills/skillseed/`。
- 改完后必须重装才生效: `bash scripts/install-skill.sh skillseed --tool claude --project`。
- 首次安装需重启对应工具; 之后重装实时生效 (纯复制, 无符号链接)。
- 经验库 `lessons.md` 是运行时状态, 重装自动保留 (install-skill.sh); 不进 canonical 源。

## 语言策略
- frontmatter `description` 用中文; 正文用中文。
- references 只允许一层深度 (不互相指示加载); 路径一律正斜杠; SKILL.md <500 行。

详见 README.md。

---
> Source: [ht426/skillseed](https://github.com/ht426/skillseed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
