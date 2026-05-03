## dotfiles

> - 永続的に必要な情報のみ、`docs/` など既存の適切な場所に残す

# Global AGENTS.md Instructions

## Project Organization
- 永続的に必要な情報のみ、`docs/` など既存の適切な場所に残す
- Keep only permanently useful information in `docs/` or another appropriate long-lived location

### 保存すべきもの (What to save):
- 長期的に参照する運用ルールや合意済みプラン (Long-lived operational rules and agreed plans)
- 今後の参考になる解決策 (Solutions that may be useful for future reference)
- プロジェクト固有の設定やワークフロー (Project-specific configurations and workflows)
- APIやライブラリの使用例 (API and library usage examples)
- 再発しうる問題の対処方法 (Fixes for issues likely to recur)

### 保存不要なもの (What not to save):
- パスワードや秘密鍵などの機密情報 (Passwords, secret keys, and sensitive information)
- 完全に自明な操作（例：単純なcdコマンドのみ）(Completely trivial operations like simple cd commands)
- 一時的な実行ログ、途中経過の調査メモ、使い捨てのプラン (Temporary execution logs, interim investigation notes, disposable plans)

## General Guidelines
- Always prefer editing existing files over creating new ones
- Never create temporary documentation files unless explicitly requested
- Persistent documentation in `docs/` (or an existing long-lived location) is allowed when it provides ongoing value
- Follow existing code conventions and patterns in each project
- Use TodoWrite tool to track complex tasks and provide visibility into progress
- テストを生成するときに可能な限り、Mockを使用しないでください
  - 言語別の指針: Ruby は WebMock のみ、JavaScript/TypeScript は jest.fn() や MSW は必要時のみ、Python は unittest.mock の必要最小限、Go は interface を使った最小限のモック
  - モックは外部I/Oや不安定な依存の隔離が必要な場合に限定する

## Code Generation Rules
- コード生成時はコメントを一切書かない (DO NOT write any comments when generating code)
- ユーザーから明示的に要求された場合のみコメントを追加する (Only add comments when explicitly requested by the user)
- This applies to all programming languages and file types
- ファイル末尾に改行をいれる

## File Management
- Check existing project structure before making changes
- Use appropriate search tools to understand codebase before implementing changes
- Run lint and typecheck commands after making code changes (if available)

## Security
- Never commit secrets or keys to repositories
- Follow security best practices in all implementations
- Only assist with defensive security tasks

## Git Commit Rules
- コミットメッセージは日本語で記述する (Write commit messages in Japanese)
- feat:, fix:, docs:, refactor: などのプレフィックスを付ける (Use prefixes like feat:, fix:, docs:, refactor:)
- コミット時は署名を必須にする (Require signed commits)
- Example: "feat: ユーザー認証機能を追加"

## Plan Workflow Defaults
- プランモードで `<proposed_plan>` を提示した直後に通常モードへ移った場合、ユーザーが拒否しない限り次を既定で実行する
- Save the proposed plan to a file by default after leaving Plan mode unless the user opts out
- 保存先はユーザー指定があれば最優先、指定がなければ `docs/plans/<YYYY-MM-DD>_<slug>.md` を使う
- `docs/plans/` は合意済みプランの保存先として扱う
- 保存後は Skill ツールで `/plan-review` を実行する（引数にファイルパスを渡す）
- `/plan-review` 完了後は結果を共有し、`/plan-go` はユーザーが明示的に要求した場合のみ Skill ツールで実行する（引数にファイルパスを渡す）
- ユーザーが明示的にスキップを指示した場合のみ `/plan-review` を省略する

---
> Source: [ryoh827/dotfiles](https://github.com/ryoh827/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
