## ruby-chains-game

> 物理カードゲーム「Ruby Chains」のプレイ補助Webアプリ。

# Ruby Chains シミュレーションアプリ - 開発引き継ぎ

## プロジェクト概要

物理カードゲーム「Ruby Chains」のプレイ補助Webアプリ。
スマホ1台を回して、メソッドチェインの実行結果やエラーを確認する判定補助ツール。

## 技術スタック

- Rails 8.1.3 / Ruby 4.0.2 / PostgreSQL
- Tailwind CSS v4 / Hotwire (Turbo + Stimulus) / importmap
- RSpec + FactoryBot（テスト）
- サンドボックスで実際にRubyコードを実行（セキュリティ対策済み）

## 現在のブランチ

`feature-dev`（main から作成、未プッシュ）

### コミット履歴

```
3ef42a9 T10: Switch to RSpec and add security tests
1034661 T5-T9: Implement full game UI with chain builder and execution
5007830 T4: Implement chain execution engine with sandboxed Ruby
c5c6d79 T3: Define card data in YAML with PORO loader classes
05ec603 T2: Add Game and GamePlayer models with validations
a4a5e1d T1: Initialize Rails 8 project with PostgreSQL and Tailwind
```

## 実装済みの機能

### バックエンド
- **Game / GamePlayer モデル**: LP/KPカウンター、2〜4人対応
- **カードデータ**: YAML定義（`config/game_data/`）、PORO（`app/models/card_data/`）
  - メソッドカード 29種（A:主力, B:ユーティリティ, C:ジェネレーター, D:切り札）
  - レシーバーカード 10種
- **チェイン実行エンジン** (`app/services/`):
  - `CodeBuilder`: カードID → Rubyコード文字列生成
  - `SandboxRunner`: 別プロセスでRuby実行（タイムアウト3秒、File/IO/system等ブロック）
  - `ChainExecutor`: ステップごと実行、中間結果収集
- 切り札: 単体（identity block）/ Proc引数（`&:method`）両対応

### フロントエンド
- ゲーム作成画面（プレイヤー数・名前・初期LP設定）
- メイン画面:
  - LP/KPカウンター（AJAX ±操作）
  - レシーバー選択ドロップダウン
  - チェイン構築UI（カード選択、切り札モーダル）
  - 実行ボタン → JSON API → 結果/エラー表示（中間結果折りたたみ）

### テスト
- RSpec 41 examples, 0 failures
- ガイドライン準拠: 日本語it/context、コントローラースペック禁止、AAAパターン
- セキュリティテスト: system/exec/File.read/require等のブロック確認

## 未完了・TODO

1. **git push**: `feature-dev` が未プッシュ（403エラー発生中）
2. **Bundler**: 環境のBundler 4.0.9にCGIバグあり。`bundle _2.5.23_` で回避中
3. **デプロイ準備** (T11): Kamaly向けDockerfile調整、credentials設定
4. **システムスペック**: Capybaraによるe2eテストは未作成

## コマンド

```bash
# DB初期化
bin/rails db:drop db:create db:migrate

# テスト実行
bundle exec rspec

# 開発サーバー（要PostgreSQL起動）
service postgresql start
bin/rails server

# プッシュ
git push -u origin feature-dev
```

## 設計ドキュメント

- `docs/tasks.md`: 実装タスク分割（T1〜T11）

## テストガイドライン

- spec は日本語で describe/context/it を記述
- コントローラースペック禁止（システムスペックで代替）
- AAA パターン、期待値はリテラル値
- 詳細は会話履歴の RSpec テストガイドラインを参照

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ynitami) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
