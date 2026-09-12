## syncgame

> ブラウザ 2D アクションゲーム（TypeScript + Canvas、通信なし）。

# SyncGame — 作業ルール

ブラウザ 2D アクションゲーム（TypeScript + Canvas、通信なし）。

## プロセス

要件定義 → 基本設計 → 詳細設計 → テスト作成 → 実装 → 修正改善 の順で進める。
定義は [docs/00_process/workflow.md](docs/00_process/workflow.md)。

**現在のフェーズ: テスト作成**

- 要件定義: 2026-08-17 確定（REQ-001〜043 / NFR-001〜006）。
  [docs/10_requirements/requirements.md](docs/10_requirements/requirements.md) を正とする
- 基本設計: 2026-08-19 完了。[docs/20_basic_design/](docs/20_basic_design/)
- 詳細設計: 2026-08-19 完了。全 MOD の公開 IF とバランス値が確定。
  [docs/30_detailed_design/](docs/30_detailed_design/)

先のフェーズの作業を勝手に始めない。実装フェーズに入る前にコードを書かない
（`scratch/` での技術検証プロトタイプは例外）。

### フェーズ4〜5 で使うスキル

順番を飛ばさない。手癖で書き始めずに該当スキルを読む。
ID 発番規則・期待値の書き方・自動テストとプレイテストの線引き・保留の判断基準が
そこに集約してある。

| フェーズ | スキル | 成果物 |
| --- | --- | --- |
| 4 前半 | `/test-cases` | `docs/40_test/cases/` の TC-xxx 項目表（日本語） |
| 4 後半 | `/test-code` | `tests/` の Vitest / Playwright テスト（全て落ちる状態） |
| 5 | `/implement` | `src/` の実装（全テスト green、通らないものは保留として記録） |

実装フェーズでテストを書き換えて通すのは禁止。テストが間違っていると判断した場合は
実装で辻褄を合わせず、`/implement` の保留手順に従って止める。

## アーキテクチャ上の絶対ルール

1. `src/domain/` は Canvas・DOM・ブラウザ API に一切依存しない。純粋ロジックのみ。
2. `src/domain/` と `src/core/` で `Math.random()` / `Date.now()` / `performance.now()` を
   直接呼ばない。乱数はシード付き、時間は引数の `dt` で受け取る。
   → 同じ入力列で同じ結果になること（決定論性）がテストの前提。
3. `src/render/` は状態を読むだけ。ゲーム状態を書き換えない。
4. バランス調整値は `src/config/` に集約。コード中に直書きしない。
5. `scratch/` を本番コードから import しない。

## ドキュメント

- 要件・テストには ID を振る（`REQ-001` / `TC-001`）。
- 要件を足したり変えたりしたら [docs/40_test/traceability.md](docs/40_test/traceability.md) を更新する。
- 後戻りコストの大きい判断は `docs/10_requirements/decisions/` に ADR として残す。

---
> Source: [compass505/SyncGame](https://github.com/compass505/SyncGame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
