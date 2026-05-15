## road-drawing

> 図面生成コードの統合リポジトリ。Excel/CSVから路線展開図・区画線展開図のDXFを生成する。

# road-drawing

図面生成コードの統合リポジトリ。Excel/CSVから路線展開図・区画線展開図のDXFを生成する。

## Architecture

```
crates/
├── dxf-engine/     — DXFエンティティ生成・バリデーション・Lint
├── road-section/   — 路線展開図ジオメトリ計算
├── road-marking/   — 区画線生成（Phase 3で追加予定）
└── excel-parser/   — Excel入力パース（Phase 2で追加予定）
cli/                — road-drawing CLIツール（AIスキル層）
web/                — egui WASMアプリ（Phase 4で追加予定）
```

## Build

```bash
cargo build
cargo test
```

## CLI Usage

```bash
road-drawing generate --input data.csv --output output.dxf --type road-section
```

## Development Process

### Agent Teams 構成
- `.claude/agents/` にレビュワー定義
- `.claude/skills/` に3つの開発スキル:
  - `plan-and-review`: 計画→AIレビュー（サブエージェント）
  - `team-impl`: 実装→レビュー→修正サイクル（Agent Teams）
  - `team-review-fix`: 人間レビュー指摘の並列対応（Agent Teams）

### Team Leader は作業を持たない
リーダーの禁止事項: コード実装、コード調査、コードレビュー、テスト実行。
リーダーの仕事: タスク割り当て、進捗管理、ユーザーとの窓口。

### 共有ファイルでタスク可視化
- `status-board.md`: メンバーのステータス・担当ファイル
- `task-log.md`: タスク一覧・担当・結果

## Rules

- DXF出力は必ず `DxfLinter::is_valid()` で検証
- テストは `cargo test` 全パスが必須
- スケール変換は m→mm (×1000) で統一
- 測点名ラベルは DXF color 5 (blue)
- GitHub Issues に対応するブランチで作業し、PRを作成

---
> Source: [YuujiKamura/road-drawing](https://github.com/YuujiKamura/road-drawing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
