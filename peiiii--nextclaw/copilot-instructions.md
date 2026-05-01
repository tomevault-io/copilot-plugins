## commit-message-english

> Use English for git commit messages


# Commit Message Language

All git commit messages must be written in **English**.

- Use conventional commit format: `type(scope): subject`
- Keep subject line under 72 characters
- Body and footer in English

```bash
# ✅ GOOD
git commit -m "fix(ncp): resolve stream encoder tool call ordering"
git commit -m "feat(api): add pagination support"

# ❌ BAD
git commit -m "修复 NCP 流式编码器问题"
git commit -m "新增分页功能"
```

---
> Source: [Peiiii/nextclaw](https://github.com/Peiiii/nextclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
