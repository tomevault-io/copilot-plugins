## evaldataset

> このファイルはClaude Codeがこのリポジトリで作業する際のガイダンスを提供します。

# CLAUDE.md

このファイルはClaude Codeがこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

HuggingFace CPT（Continued Pre-Training）用データセットの品質チェック・自動修正を行うPython CLIアプリケーション。

## 最重要事項

- GitHubフロー
- `docs/`のドキュメントを参照
- TDD（テスト駆動開発）
- pytest による単体テスト、カバレッジ計測
- パッケージ管理は uv を使用

## 技術スタック

- Python 3.11+
- Click（CLIフレームワーク）
- datasets（HuggingFace）
- Rich（コンソール出力）
- pytest + pytest-cov（テスト・カバレッジ）

## コマンド

```bash
# セットアップ
uv sync --dev

# テスト実行
uv run pytest

# カバレッジ付きテスト
uv run pytest --cov=evaldataset --cov-report=term-missing

# CLI実行
uv run evaldataset <DATASET_ID> [OPTIONS]
```

## プロジェクト構造

- `src/evaldataset/` - メインソースコード
- `tests/unit/` - ユニットテスト（src参照OK、モックOK）
- `tests/integration/` - 結合テスト（specベース、モック禁止）
- `configs/` - 設定ファイル
- `docs/` - ドキュメント

## テスト戦略

UT（ユニットテスト）とIT（結合テスト）を分離する。

### UT（ユニットテスト）

- **配置**: `tests/unit/`
- **src参照**: OK
- **モック**: OK（`docs/mock-policy.md` 参照）
- **作成（手動）**: `/unit-test` スキル
- **作成（ループ）**: Agent Teams の test-writer（`/continue-task` `/fix-issues` から起動）
- **実行**: `uv run pytest tests/unit/ -v`

### IT（結合テスト）

- **配置**: `tests/integration/`
- **src参照**: 禁止（`docs/design.md` の受入条件から導出）
- **モック**: 正常系では禁止（`docs/mock-policy.md` 参照）
- **作成（手動）**: `/spec-test` スキル（コンテキスト分離）
- **作成（ループ）**: Agent Teams の it-writer（`/continue-task` `/fix-issues` から起動）
- **実行**: `uv run pytest tests/integration/ -v`
- **トレーサビリティ**: 各テストにGiven/When/Thenコメント必須

### 設計書のspec化ルール

`docs/design.md` に新しいチェッカーやfeatureを追加する際は、Given/When/Then形式の受入条件を必ず記述する。この受入条件がITの導出元となる。

## Agent Teams

タスク実装・レビューにAgent Teamsを活用する。

### エージェント一覧（`.claude/agents/`）

#### 調査・設計チーム

| エージェント | 役割 | 使用タイミング |
|-------------|------|--------------|
| researcher | 検証手法の調査 | `/research-design` でchallenger/spec-writerと並列 |
| challenger | 弱点・ギャップ指摘 | `/research-design` でresearcher/spec-writerと並列 |
| spec-writer | 受入条件（spec）作成 | `/research-design` で議論収束後に動作 |

#### 実装チーム

| エージェント | 役割 | 使用タイミング |
|-------------|------|--------------|
| implementer | コード実装 | フェーズ2でtest-writerと並列 |
| test-writer | UT作成 | フェーズ2でimplementerと並列 |
| it-writer | IT作成（src参照禁止） | フェーズ3で単独 |
| fixer | レビューMajor指摘の修正 | フェーズ4.5で単独 |

#### レビューチーム

| エージェント | 役割 | 使用タイミング |
|-------------|------|--------------|
| code-reviewer | コードレビュー | フェーズ4でut/it-validatorと並列 |
| ut-validator | UT品質検証 | フェーズ4でcode-reviewer/it-validatorと並列 |
| it-validator | IT品質検証 | フェーズ4でcode-reviewer/ut-validatorと並列 |

### ワークフロー

```
フェーズ1: /research-design（調査・設計チーム、spec作成）
    ↓    または /create-spec（簡易版、specのみ作成）
フェーズ2: implementer + test-writer（並列、UT作成）
    ↓
フェーズ3: it-writer（IT作成、src参照禁止）
    ↓
フェーズ4: code-reviewer + ut-validator + it-validator（並列レビュー）
    ↓
フェーズ4.5: fixer（Major指摘があれば修正）
```

### オーケストレータ / ワーカー分離

`/continue-task` `/fix-issues` ではオーケストレータ/ワーカー分離を採用する:

- **オーケストレータ（メイン会話）**: ループ制御、git操作、Agent起動、テスト結果確認のみ。ソースコードを直接読まない・書かない
- **ワーカーAgent**: 実装・テスト・レビュー・修正の全てを担当

この分離により、オーケストレータのコンテキストが軽量に保たれ、ループ指示が圧縮されにくくなる。

### ファイル競合の回避

- implementer → `src/` 配下を担当
- test-writer → `tests/unit/` 配下を担当
- it-writer → `tests/integration/` 配下を担当
- fixer → 全ファイル編集可能（レビュー指摘の修正のため）
- 同一ファイルを複数teammateが編集しないよう、リードが調整する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YasudaHispot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
