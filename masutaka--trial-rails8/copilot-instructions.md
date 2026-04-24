## trial-rails8

> - MVC アーキテクチャを遵守し、ビジネスロジックはモデル層に配置すること

# CLAUDE.md

## 基本原則

- MVC アーキテクチャを遵守し、ビジネスロジックはモデル層に配置すること
- コントローラは薄く保ち、リクエスト/レスポンスの処理に専念すること
- ジョブは薄く保ち、非同期処理のオーケストレーションに専念すること

## TDD での注意点

- テストはファイル単位で実行する
    - 例: "rails test test/models/post_test.rb"
- ルーティングのテストも書く
    - test/routing/

## ドキュメント更新での注意点

- 既存のテキストを更新することで、ドキュメントのボリューム（行数）が増えないようにする。変更履歴のような追記をしないこと

## コミット前の注意点

- "bin/rubocop" のエラーがないことを確認すること
    - エラーが発生したら "bin/rubocop -a" で修正を試みること

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masutaka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
