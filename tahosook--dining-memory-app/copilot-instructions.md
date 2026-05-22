## dining-memory-app

> Codex がこの repo で毎回同じ前提を使い、必要以上に context を消費しないための固定ルールです。

# AGENTS.md

## Purpose
Codex がこの repo で毎回同じ前提を使い、必要以上に context を消費しないための固定ルールです。
詳細な読み先は [docs/engineering/codex-context-map.md](docs/engineering/codex-context-map.md) を参照してください。

## Read First
1. [docs/engineering/codex-context-map.md](docs/engineering/codex-context-map.md)
2. 今回のタスクに必要な docs / source / tests だけ

毎回 repo 全体、`docs/` 全体、`src/` 全体を読まないでください。
`TASKS.md` は作業候補、`PLANS.md` は大きめ作業の計画です。該当タスクのときだけ参照します。

## Working Rules
- 編集前に、読んだファイル、編集予定ファイル、実装計画を短く整理する。
- 大きな作業は「調査」「計画」「実装」に分ける。
- 変更は最小差分にする。
- 関係ない改善、大規模リファクタ、ついでの整形をしない。
- 既存の設計方針や user changes を勝手に置き換えない。
- 実装変更では、近い既存パターンと関連テストを確認してから編集する。
- product / UX / data / engineering convention が変わる場合は、対応する canonical doc も同じタスクで更新する。

## AI / MediaPipe Rules
- food-labeling、MediaPipe、AI labeling に関係する作業では、まず [docs/engineering/food-labeling-guidelines.md](docs/engineering/food-labeling-guidelines.md) を最優先で尊重する。
- MediaPipe path は app 全体の default runtime や永続化契約へ勝手に昇格しない。
- AI 入力補助は optional で、manual save を妨げない。
- raw AI output、model metadata、photo path、location、notes などの sensitive data を不要に保存・ログ出力しない。

## Finish Report
作業後は以下を簡潔に報告してください。
- 変更ファイル
- 確認コマンド
- 未確認事項
- テストや lint を実行しなかった場合はその理由

---
> Source: [tahosook/dining-memory-app](https://github.com/tahosook/dining-memory-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
