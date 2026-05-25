## bkm-old

> - **目的**: bookmarkをfuzzy

# Deno Bookmark CLI Tool 開発指針

## プロジェクト概要

- **目的**: bookmarkをfuzzy
  finderで開き、選択したbookmarkをブラウザで開くCLIツール
- **技術**: Deno、TypeScript
- **データ保存**: `~/.local/share/bkm/`
- **bookmark情報**: タイトル、URL、タグ（複数設定可能）

## アーキテクチャ

- **Clean Architecture** + **DDD (Domain-Driven Design)**
- **4層構造**:
  - `cli/`: 入出力層（CLI interface）
  - `core/`: ドメイン層（Entity, Value Object, Repository Interface）
  - `gateway/`: ゲートウェイ層（Repository実装、外部システム連携）
  - `usecase/`: ユースケース層（ビジネスロジック）

## ディレクトリ構成

```
src/
├── cli/                    # 入出力層
│   ├── commands/          # CLI command handlers
│   ├── prompts/           # Interactive prompts
│   └── main.ts            # CLI entry point
├── core/                   # ドメイン層
│   ├── bookmark/          # Bookmark ドメイン
│   ├── config/            # Configuration ドメイン
│   └── shared/            # 共通（ValueObject interface）
├── gateway/               # ゲートウェイ層
│   ├── storage/           # Repository implementations
│   ├── browser/           # Browser opening
│   └── fuzzy/             # Fuzzy finder integration
└── usecase/               # ユースケース層
    ├── bookmark/          # Bookmark operations
    ├── config/            # Configuration management
    └── search/            # Search and filtering
```

## 技術スタック

- **CLI**: `@cliffy/command@1.0.0-rc.8`, `@cliffy/prompt@1.0.0-rc.8`
- **エラーハンドリング**: `@praha/byethrow@0.6.0` (Result型)
- **テスト**: `fast-check@4.2.0` (Property-Based Testing)
- **ランタイム**: Deno

## CLI コマンド仕様

```bash
# 対話型でブックマーク追加
bkm add

# オプション指定でブックマーク追加
bkm add <url> --title "タイトル" --tags "tag1,tag2"

# ブックマーク編集
bkm edit

# ブックマーク削除
bkm remove

# fuzzy finder設定
bkm config fuzzy --command "fzf" --args "--height 40%"

# デフォルト動作: fuzzy finderでブックマーク一覧表示
bkm
bkm --tag "tag1,tag2"  # タグフィルタリング
```

## 開発方針

### 1. TDD (Test-Driven Development) - 和田卓人（t_wada）のTDD

- **Red-Green-Refactor**サイクル
- **Red**: 失敗するテストを最初に書く
- **Green**: テストを通すための最小限の実装
- **Refactor**: 動作するコードを改善

### 2. Test Pyramid

- **Unit Tests (多数)**: Value Object, Entity, Use Case
- **Integration Tests (中程度)**: Repository, Gateway層
- **E2E Tests (少数)**: CLI Commands

### 3. Property-Based Testing

- **fast-check**を使用してPBTを原則とする
- **テストファイル**: 実装ファイルと同階層に `*.test.ts` で配置

### 4. Result型による安全なエラーハンドリング

- すべての操作をResult型で包む
- パイプライン式の処理で可読性向上

### 5. Value Object設計

- **共通インターフェース**: `ValueObject<T>`
- **必須メソッド**: `equals()`, `toString()`
- **不変性**: 値の変更不可
- **等価性**: 値による比較

## 実装順序

1. ValueObject interface
2. BookmarkId value object
3. BookmarkTitle value object
4. BookmarkUrl value object
5. BookmarkTag value object
6. Bookmark entity
7. Use Cases
8. Repository implementations
9. CLI Commands

## コミット戦略

- **適切な粒度**: 1つのValue Object/Entity/Use Caseごと
- **TDDサイクル**: Red-Green-Refactorの完了時点
- **コミットメッセージ例**:
  - `feat: add BookmarkId value object`
  - `test: add property-based tests for BookmarkUrl`
  - `refactor: improve BookmarkTitle validation`

## 制約・注意点

- **TDDを厳守**: テストファーストで実装
- **Result型必須**: エラーハンドリングは必ずResult型
- **PBT原則**: fast-checkを使用した特性ベーステスト
- **コミット粒度**: 適切な単位で頻繁にコミット
- **Clean Architecture**: 依存関係の方向を守る

---
> Source: [airRnot1106/bkm-old](https://github.com/airRnot1106/bkm-old) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
