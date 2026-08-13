## review-lens

> Devin Review的なAIコードレビュー体験をローカルで実現するデスクトップアプリ。Claude Code CLIをサブスクリプションで利用し、APIコストを回避する。

# review-lens

Devin Review的なAIコードレビュー体験をローカルで実現するデスクトップアプリ。Claude Code CLIをサブスクリプションで利用し、APIコストを回避する。

## ディレクトリ構造

```
review-lens/
├── src/              # React フロントエンド
├── src-tauri/        # Rust バックエンド (Tauri v2)
├── AGENTS.md
├── CLAUDE.md
└── plan.md           # 開発方針・フェーズ計画
```

## セットアップ

```bash
# 前提: Node.js, Rust, Tauri CLI
pnpm install
pnpm tauri dev
```

## 技術スタック

- **フレームワーク**: Tauri v2
- **フロントエンド**: React + TypeScript
- **UIコンポーネント**: shadcn/ui
- **バックエンド**: Rust (subprocess管理中心)
- **AI**: Claude Code CLI (`claude -p`)
- **GitHub連携**: gh CLI

## コミュニケーション方針

- 忖度しない。問題点やリスクがあれば率直に指摘する
- コメントは「なぜ」を説明する場合にのみ書く

## Ralph Workflow

- Ralph runtime files live under `.agent/ralph/`
- Main state files are `.agent/ralph/prd.json` and `.agent/ralph/progress.txt`
- Each Ralph iteration should work on exactly one story
- A story is not done until required quality checks pass
- The final story in `.agent/ralph/prd.json` must be a Codex review and remediation story
- The review story is not done until Codex review is complete, every finding is fixed, and required quality checks pass again
- After finishing a story, update `.agent/ralph/prd.json` and append progress to `.agent/ralph/progress.txt`
- Commit completed work with `feat: [Story ID] - [Story Title]`
- Reusable learnings should be written back to the nearest relevant `AGENTS.md`
- UI changes should be browser-verified using `agent-browser skill`
- Reply with `<promise>COMPLETE</promise>` only when every story has `passes: true`

---
> Source: [wwwyo/review-lens](https://github.com/wwwyo/review-lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
