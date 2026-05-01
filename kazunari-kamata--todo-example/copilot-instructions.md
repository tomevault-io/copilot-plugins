## todo-example

> このリポジトリには repo-local skill として `.codex/skills/github-review-workflow/` があります。

# Repository Agents

## Repo-Local Skill

このリポジトリには repo-local skill として `.codex/skills/github-review-workflow/` があります。

次の作業では `$github-review-workflow` を使ってください。

- GitHub Actions workflow の追加や修正
- CI や review 設定に合わせた README 更新
- `gitleaks` hook の設定や修正
- `.github/CODEOWNERS` の追加や更新
- pull request 作成と GitHub checks の確認
- branch protection の整合調整

## Notes

- 追跡されている workflow は `git ls-tree --name-only -r HEAD .github/workflows` で確認します。
- ローカルにあるファイルが `git status` に出ない場合は `git check-ignore -v <path>` で ignore 設定を確認します。
- この環境では `.github/...` や `.codex/...` のような hidden path に `git add -f` が必要になることがあります。

---
> Source: [kazunari-kamata/todo-example](https://github.com/kazunari-kamata/todo-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
