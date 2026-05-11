## terminal-session-recall

> > 汎用ルール（行動規範、git運用、Dependabotポリシー、環境注意事項等）は `~/.claude/CLAUDE.md` を参照（chezmoi管理）。

# Claude Resurrect — Claude Code 設定

> 汎用ルール（行動規範、git運用、Dependabotポリシー、環境注意事項等）は `~/.claude/CLAUDE.md` を参照（chezmoi管理）。

## プロジェクト概要

VSCode 再起動時に Claude Code CLI セッションを自動復元する拡張機能。

## 開発コマンド

```bash
npm run typecheck  # 型チェック
npm run test       # Vitest テスト
npm run compile    # ビルド（esbuild で out/ に出力）
npm run watch      # ファイル変更監視 + 自動ビルド
npm run package    # .vsix パッケージ作成
```

**品質ゲート**（作業完了前に必ず実行）: `npm run typecheck && npm run test && npm run compile`

## アーキテクチャ

### ~/.claude/ アクセス制約

- `claude-dir.ts` が唯一の `~/.claude/` アクセスモジュール
- 他のモジュールから fs で直接 `~/.claude/` に触ることを禁止
- 読み取り専用のみ。書き込み API は一切使用しない

### テスト可能ファイル

| ファイル | Vitest テスト可能 | 理由 |
|---------|:-:|------|
| claude-dir.ts | Yes | fs のみ依存。vscode API 非依存 |
| session-store.ts | Yes | persist コールバックで DI 済み |
| normalize-path.ts | Yes | 純粋関数 |
| extension.ts | No | vscode API 依存。F5 デバッグで検証 |

## コーディングスタイル

- TypeScript strict
- イミュータブル操作（spread で新オブジェクト作成）
- `any` 禁止（`unknown` を使う）

---
> Source: [orangewk/terminal-session-recall](https://github.com/orangewk/terminal-session-recall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
