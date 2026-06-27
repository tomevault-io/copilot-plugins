## git-commit

> 当用户要求快速生成 git commit message、自动提交当前变更、拆分提交、使用 feat/fix/docs 等 Conventional Commits 前缀，或调用 git-commit 时使用。


# Git Commit

优先读取 `git-commit/SKILL.md`；不可读时按此最小流程执行。

1. 先跑 `git status --short`；空则回复 `nothing to commit`。
2. 有 staged changes 时只处理 staged，除非用户要求全部变更。
3. 先看 `git diff --cached --name-status`/`--stat`；无 staged 时看 `git diff --name-status`/`--stat`。
4. 只有 type 或 summary 不明确时才读精准 diff：`git diff --cached -- path` 或 `git diff -- path`。
5. 相关小改动合并；不同目的、不同 type、或 message 会变泛时拆成多个 commit。
6. Message 必须是英文、单行、祈使语气、30 词以内，格式 `type: summary`。
7. `type` 只选 `feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`style`、`perf`、`ci`、`build`、`revert`。
8. 每次只 stage 一个提交组，核对 cached name-status/stat 后运行 `git commit -m "type: summary"`。
9. 不使用 `git reset --hard`、`git checkout --`、amend、rebase、force push，不覆盖用户变更。

---
> Source: [yangin/ai-skills](https://github.com/yangin/ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
