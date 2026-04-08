## kavukiai

> - [doc](doc)folderにフォルダ構造など開発者向け文章あり

# Contributor Guide

## Dev Environment Tips
- [doc](doc)folderにフォルダ構造など開発者向け文章あり
- PowerShellで日本語が文字化けする場合は、ターミナル起動後に $OutputEncoding = [Console]::OutputEncoding = [System.Text.Encoding]::UTF8 を実行してから "Get-Content" などのコマンドを使うと読みやすくなります。必要ならプロファイルに設定を追記してください。

## Refer to plan
When I say refer to <planname> plan, open doc/plan/<planname>-plan.md (for example doc/plan/node-refactor-plan.md) and proceed. If you get confused during the process, refer to it each time.

## Testing Instructions
- From the package root you can just call pnpm test run. The commit should pass all tests before you merge.
- To focus on one step, add the Vitest pattern: pnpm test run -t "<test name>".
- Fix any test or type errors until the whole suite is green.
- After moving files or changing imports, run pnpm lint .
- Add or update tests for the code you change, even if nobody asked.

## PR
Before PR, pull the latest version and check it with pnpm test run, pnpm lint.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahehead)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ahehead)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
