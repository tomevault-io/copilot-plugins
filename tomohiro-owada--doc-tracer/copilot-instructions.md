## doc-tracer

> このファイルは、doc-tracerプロジェクトでClaude Codeが作業する際の指針を提供します。

# CLAUDE.md

このファイルは、doc-tracerプロジェクトでClaude Codeが作業する際の指針を提供します。

---

## プロジェクト概要

doc-tracerは、**ドキュメントとコードの双方向トレーサビリティを実現するCLIツール**です。

### 解決する問題

1. **ドキュメントとコードの乖離**: コード変更後のドキュメント更新漏れ
2. **影響範囲の不明確さ**: どのドキュメントを更新すべきか分からない
3. **AIによるドキュメント更新の困難さ**: Claude Codeがどのドキュメントを更新すべきか判断できない

### 核心的なアイデア

- ドキュメントとコードの関係を**有向グラフ**として表現
- YAMLフロントマター（手動）と静的解析（自動）の組み合わせ
- **影響範囲の逆引き**: コード変更 → 更新すべきドキュメントを自動特定

---

## 技術スタック

- **言語**: Go 1.21+
- **CLI**: [Cobra](https://github.com/spf13/cobra)
- **DB**: SQLite (modernc.org/sqlite - Pure Go)
- **Web UI**: D3.js (埋め込みHTML)

---

## ディレクトリ構成

```
doc-tracer/
├── CLAUDE.md           # このファイル（Claude Code向け指針）
├── README.md           # 設計思想・概念説明
├── main.go             # エントリポイント
├── cmd/
│   ├── root.go         # ルートコマンド
│   ├── scan.go         # スキャンコマンド（コード解析）
│   ├── serve.go        # Webサーバー
│   ├── check.go        # 整合性チェック
│   ├── impact.go       # 影響範囲検索
│   └── web/
│       └── index.html  # Web UI
├── internal/
│   └── db/
│       └── schema.go   # DBスキーマ・CRUD
├── .claude/
│   └── skills/         # Claude Code用スキル
│       ├── doc-tracer-scan/
│       ├── doc-tracer-add-frontmatter/
│       ├── doc-tracer-update-docs/
│       └── doc-tracer-check/
├── go.mod
└── go.sum
```

---

## 主要コマンド

| コマンド | 説明 |
|---------|------|
| `doc-tracer scan [path]` | ドキュメントとコードをスキャン |
| `doc-tracer serve [--port]` | Web UIを起動 |
| `doc-tracer check` | 整合性チェック |
| `doc-tracer impact <file>` | 影響範囲を検索 |

---

## 5層ドキュメント構造

doc-tracerが想定するドキュメント階層:

```
01_要件定義/    # WHY - なぜ作るのか
02_仕様書/      # WHAT - 何を作るのか
03_基本設計/    # HOW（概要）
04_詳細設計/    # HOW（詳細）
05_プログラム設計/  # 実装設計
src/            # コード
```

### 層間の関係

- `contains`: 親 → 子（階層構造）
- `implements`: 実装 → 要件
- `uses`: 参照関係
- `calls`: API呼び出し

---

## YAMLフロントマター

```yaml
---
trace:
  id: doc:docs/03_基本設計/認証設計書.md
  parent:
    - doc:docs/02_仕様書/認証仕様書.md
  children:
    - doc:docs/04_詳細設計/ログイン詳細設計.md
  uses:
    - component:LoginForm
---
```

### IDの命名規則

| プレフィックス | 対象 |
|--------------|------|
| `doc:` | ドキュメント |
| `component:` | Vueコンポーネント |
| `api:` | APIエンドポイント |
| `controller:` | PHPコントローラー |
| `model:` | Eloquentモデル |
| `store:` | Piniaストア |
| `db:` | DBテーブル |

---

## Claude Code向けスキル

### `/doc-tracer-scan`

プロジェクトをスキャンしてトレーサビリティグラフを構築。
新規プロジェクトで最初に実行。

### `/doc-tracer-add-frontmatter`

新規ドキュメントにYAMLフロントマターを追加。
5層構造に基づいて適切なparentを設定。

### `/doc-tracer-update-docs`

コード変更後に影響を受けるドキュメントを特定・更新。
`doc-tracer impact` を使って影響範囲を調査。

### `/doc-tracer-check`

整合性チェック。リンク切れ・孤立ドキュメントを検出して修正。

---

## 開発ガイドライン

### コード変更時

1. 変更を実装
2. `doc-tracer impact <file>` で影響範囲を確認
3. 影響を受けるドキュメントを更新
4. `doc-tracer check` で整合性確認
5. コミット

### ドキュメント追加時

1. 適切な層にドキュメントを作成
2. YAMLフロントマターを追加（parent設定）
3. `doc-tracer scan docs --docs-only` で反映
4. `doc-tracer check` で孤立チェック

### ビルド

```bash
go build -o doc-tracer .
```

### テスト

```bash
go test ./...
```

---

## 注意事項

1. **parentとchildren**: どちらを使っても同じエッジが生成される。子ドキュメント側にparentを書く方が運用しやすい。

2. **idフィールド**: 省略可能。スキャン時にファイルパスから自動生成される。

3. **除外パターン**: `node_modules/`, `vendor/`, `.git/`, `99_アーカイブ/`, `90_作業ログ/` は自動的に除外。

4. **シンボリックリンク**: `realpath` で解決してからスキャン。

---
> Source: [tomohiro-owada/doc-tracer](https://github.com/tomohiro-owada/doc-tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
