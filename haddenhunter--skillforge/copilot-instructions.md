## skillforge

> - Rust: `cargo build --release -p skillforge-core` / `cargo test`

# AGENTS.md — SkillForge

## Build & Verify
- Rust: `cargo build --release -p skillforge-core` / `cargo test`
- Registry: `pnpm --filter registry install && pnpm --filter registry test`
- Lint: `cargo clippy --all-targets` + `pnpm biome check`
- Done 门禁：test 全绿 + clippy 零警告 + 涉及 skill 改动必须有 eval.md CI 通过

## Repo Layout（违反即打回）
- core/        → Rust：CLI、沙箱、agent loop、skill.yml 校验
- registry/    → TS：注册表 API、SQLite、publish/install
- skills/<name>/ → skill.yml + SKILL.md + eval.md + scripts/ + references/

## Hard Constraints
- 沙箱默认 deny-all；allow 白名单外禁止 fs/net/exec 提权
- skill.yml 禁止私有字段；schema 改动需同步 core 校验器 + 示例 skill
- 不引 LangChain / Dify / n8n；CLI 零 Python 运行时依赖
- 模型调用必须过 token 预算截断
- commit 用 conventional commits；不改未授权工作区文件

## MCP
- `skillforge serve <skill>` → localhost:18080/mcp，供 Claude Code/Cursor/Codex 反向调用
- 接外部 MCP 时默认只读，写操作留人工确认

## Skills
- 重复 2+ 次的操作提炼成 skills/<name>/，description 写清触发边界（progressive disclosure）
- 存 $REPO_ROOT/.agents/skills 或 skills/，二者保持同步

---
> Source: [HaddenHunter/SkillForge](https://github.com/HaddenHunter/SkillForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
