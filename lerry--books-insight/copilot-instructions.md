## books-insight

> 这个仓库是 `books-insight` 的公开 prompt 仓库，不承载业务代码。

# Prompt-Only Repository

这个仓库是 `books-insight` 的公开 prompt 仓库，不承载业务代码。

先读这些规格：

- `specs/backend-spec.md`
- `specs/search-spec.md`
- `specs/homepage-spec.md`

默认提示词入口：

- 一把生成：`prompts/00-one-shot-full.md`
- 分步生成：`prompts/10-backend-only.md` -> `prompts/20-homepage-only.md` -> `prompts/30-verify.md`
- 默认实战测试：`prompts/40-default-eval-live-api.md`

额外说明：

- `prompts/90-prompt-lineage.md` 只用于帮助理解需求来源
- 真正编码时，以 `specs/*.md` 为准

边界：

- 这个仓库只放 prompt 和规格
- 不要往这个仓库里补业务代码
- 真正的代码应该生成到目标项目里

---
> Source: [lerry/books-insight](https://github.com/lerry/books-insight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
