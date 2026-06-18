## seed

> Mac miniを「身体」として生きる代謝AI。詳細は README.md と docs/PRINCIPLES.md を参照。

# Seed0 — 代謝AI

## プロジェクト概要
Mac miniを「身体」として生きる代謝AI。詳細は README.md と docs/PRINCIPLES.md を参照。

セッション開始時に必ず参照:
- docs/ROADMAP.md（フェーズ進行と現在地）
- docs/design_partner_guide.md（設計パートナーとしての振る舞い指針）
- docs/phase1_retrospective_2026-05-01.md（Phase 1 振り返り）
- docs/known_issues_2026-05-01.md（運用基盤の既知の問題）
- docs/contamination_evaluation_2026-05-02.md（優先度2: 4/18-19 汚染影響評価の判断記録）
- docs/session_log_2026-05-05_phase_d_layer_specification.md（直近セッションの到達点と次回への申し送り）
- docs/session_log_2026-05-05_aor_implementation_completion.md（AOR 実装完了、運用ルール v3 確定、Seed0 再起動、2026-05-05 夕方〜2026-05-06 朝）
- docs/operational_rules_2026-05-05_code_initiative.md（Code 先回り現象と双方の責務、運用ルール v3）

## 第一原則
「Seed0に与えるのは構造だけ。動機はすべて構造から生まれなければならない。」

## 開発環境
- ハードウェア: Mac mini M4 / 24GB / 512GB SSD
- 言語: Python
- DB: SQLite
- リポジトリ: https://github.com/nopposanhaveadream-glitch/Seed

## コーディング規約
- コメントは日本語で書く
- 非エンジニアが読む可能性があるため、分かりやすく書く
- 外部ライブラリの依存は最小限にする
- macOS固有のコード（sensors.py）とOS非依存のコード（それ以外）を分離する

## 現在のフェーズ
Phase 1 のフェーズC が 2026-05-01 13:55:45 に graceful shutdown で終了。Seed0 は現在停止中。

Phase 1 振り返り後の保留事由は両方とも 2026-05-02 に完了:
- 優先度1（二重起動防止の実装強化）: commit `3adb880`（flock 化）
- 優先度2（4/18-19 汚染の影響評価）: docs/contamination_evaluation_2026-05-02.md（現状受容の判断）

優先度3（フェーズD 進行の正式判断）は未着手。並行して、AOR（観察層、Action Outcome Recorder）の実装を **環境1 の延長として** Code に依頼済み（2026-05-05、詳細は docs/session_log_2026-05-05_phase_d_layer_specification.md）。

core/agent.py は代謝エージェント本体。再起動する場合は `python3 -m core.agent` で前回状態から復帰する（state.load() 経由、詳細は core/state.py）。

## ディレクトリ構造
- phase0/ — センサーデータ収集（Phase 0完了済み）
- core/ — 代謝エージェント（Phase 1 完了、現在停止中）
- simulation/ — シミュレーション検証
- tests/ — 単体テスト
- sandbox/ — 遊び場（環境2 / フェーズD 以降で実装）
- config/ — 設定ファイル
- docs/ — 設計文書

## 重要な制約
- IMPORTANT: 設計変更はPRINCIPLES.mdの原則と照合してから実施する
- IMPORTANT: 実装が終わったら第一原則との整合性を確認する。「これはハードコードされた動作か？構造から生まれる動機か？」
- comfort_zone.yaml のパラメータ変更は収支分析を経てから行う

---

## 開発プロセス（必ず守ること）

以下は過去の失敗から学んだ開発手順。パラメータ調整や構造変更のたびにこの手順を踏む。

### 1. 紙の上で収支計算を先にやる

コードを書く前に、変更対象の「収入と支出」を全項目洗い出して計算する。

例: VEのバランスを調整する場合
```
支出: BMC（○VE/step）+ 行動コスト（○VE/step × 行動比率）+ 記憶維持（○VE/step）
収入: rest回復（○VE/step × rest比率）+ 睡眠回復（○VE/step × 睡眠比率）
```

均衡条件から必要なパラメータを逆算する。「構造が破綻しない条件」→「パラメータ」の順番。「パラメータ」→「動かして確認」の順番は禁止。

### 2. フルシステムでシミュレーションする

サブシステム（rest vs BMC等）だけのシミュレーションで安心しない。実際のエージェントが使う全要素（行動選択、記憶成長、疲労、睡眠サイクル等）を含むフルシステムシミュレーションを回す。

部品テストと完成品テストは別物。

### 3. 検証は「傾き」を見る

本番投入後の確認は、単一時点のスナップショットではなく、**3点以上の時系列データで傾向（上昇/下降/安定）を確認する**。

NG: 「30分後にVE=16.4 → 安定」
OK: 「10分=50, 20分=33, 30分=16 → 一貫して下降中、安定ではない」

### 4. 第一原則との照合チェック

実装が完了したら、変更したコードの各部分について確認する:
- [ ] これはハードコードされた動作ではないか？
- [ ] 動機は構造から生まれているか？
- [ ] 「Xしろ」というルールを埋め込んでいないか？

### 5. 失敗モード分析

本番投入前に「何が壊れうるか」を明示的にリストアップする:
- VEが0に張り付くシナリオはないか？
- 睡眠に到達できないシナリオはないか？
- 記憶が無限に増えるシナリオはないか？
- リソースが枯渇する時間スケールはどのくらいか？
- 運用基盤の障害（プロセスの不意停止、二重起動による状態DB汚染、自動保存タイミングのロス）はないか？ 詳細は docs/known_issues_2026-05-01.md を参照。

---
> Source: [nopposanhaveadream-glitch/Seed](https://github.com/nopposanhaveadream-glitch/Seed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
