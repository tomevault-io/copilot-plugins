## room-synth-sim

> 2D音場内を身体として移動し、左右耳で聴取できるRoom Synth Simulatorを構築する。

# AGENTS.md

## Mission
2D音場内を身体として移動し、左右耳で聴取できるRoom Synth Simulatorを構築する。

## Source of Truth
- docs/specification/
- 09_ACCEPTANCE_TESTS.md
- 08_MILESTONES.md

## Working Rules
- 変更前に関連仕様を読む
- 実装とテストを同一タスクで行う
- TypeScript strict
- any乱用禁止
- 音響処理はAudioWorklet
- Reactから音声ブロックを処理しない
- graph compileとruntimeを分離
- invalid cycleはcompile時拒否
- FeedbackDelayだけ循環可能
- master limiter必須
- emergency stop必須
- 履歴4レイヤーを独立保持
- live sessionは巻き戻さない
- 過去から再開時はbranch
- 設計判断はADRへ記録

## Definition of Done
implementation, unit tests, integration tests, relevant e2e, typecheck, lint, no regression, acceptance criteria verified, documentation updated。

---
> Source: [chanwata/room-synth-sim](https://github.com/chanwata/room-synth-sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
