## machikoro

> このファイルは、このリポジトリで Claude Code が最初に参照する入口です。

# CLAUDE.md

このファイルは、このリポジトリで Claude Code が最初に参照する入口です。

まずここを読み、詳細はリンク先の文書を参照してください。

## 優先事項

1. ローカル対戦、オンライン同期、再接続復元、保存状態、PWA 更新フローの挙動は一貫させてください。
2. `js/GameManager.js` をルールの唯一の正として扱ってください。
3. オンライン挙動、再接続挙動、CPU の進行タイミングは、確認なしで安易に変えないでください。

## Claude 向けメモ

- このリポジトリは Android + Termux から編集されることが多いです。
- 長いシェルコマンドは端末の折り返しで壊れやすいので、使える場合は既存のラッパースクリプトを優先してください。
- RL の baseline 学習は、フルコマンドを打ち直さず `sh scripts/rl/run-baseline.sh` を使ってください。既定値は Termux 向けに軽量化され、進捗表示も有効です。
- `termux-chroot` が有効でない場合、一部のシェル挙動は通常の Linux デスクトップと異なることがあります。

## 次に読む文書

- リポジトリ全体のルールと責務分担: [AGENTS.md](./AGENTS.md)
- 人間向けの全体概要と主要コマンド: [README.md](./README.md)
- RL 学習・評価・artifact の流れ: [scripts/rl/README.md](./scripts/rl/README.md)
- 高リスク変更時の手動確認項目: [TESTPLAN.md](./TESTPLAN.md)

## 重要な不変条件

- `index.html` の script 読み込み順は重要です。
- オンライン対戦は決定論的で、ホスト主導です。
- サーバーはアクションを検証・中継しますが、完全なゲーム状態は持ちません。
- 再接続とサーバー再起動後の復元は、例外対応ではなく製品の中核機能です。
- 文字列リテラルではなく、`CARD_EFFECTS`、`CARD_CATEGORIES`、`LANDMARK_NAMES`、`GAME_PHASES`、`LOG_TYPES` など既存定数を使ってください。

## 主な責務の対応

- ルール本体: `js/GameManager.js`
- カード・定数: `js/Card.js`
- 通常 CPU: `js/CPU.js`
- RL CPU ランタイム: `js/RLCPU.js`
- オンラインクライアントの流れ: `js/online.js`, `js/main.js`
- 保存・再接続永続化: `js/storage.js`
- UI / モーダル: `js/ui.js`
- サーバー検証 / ルーム / 復元: `server.js`
- RL 学習・評価フロー: `scripts/rl/*.py`, `scripts/*.js`, `models/rl_model/`

## よく使う確認コマンド

- 全自動テスト: `npm test`
- クライアント 1 ファイルの構文確認: `node --check js/<file>.js`
- RL baseline 学習: `sh scripts/rl/run-baseline.sh`
- RL と JS CPU の比較: `npm run eval-rl-vs-js -- --model <path>`
- RL metrics 集計: `npm run summarize-rl-metrics -- --csv models/rl_model/train_metrics.csv`

## 編集時の基本

- ルール、オンラインフロー、再接続、検証、共有ゲーム挙動を変更したら `npm test` を実行してください。
- CPU / self-play / tuning を変更したら、関連する CPU 系テストも追加で実行してください。
- RL ランタイムや学習・評価ツールを変更したら、RL 系テストも追加で実行してください。
- オンラインや再接続を変更したら、[TESTPLAN.md](./TESTPLAN.md) の手動確認項目も見てください。

## RL の位置づけ

- RL は現在 `expert` とは別ラインです。
- 製品方針としては、`expert` を直接置き換えるのではなく、新しい CPU として導入する前提です。
- checkpoint の品質判断は `vs_random` だけでなく、JS CPU 評価と summary artefact を主に見てください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nao70161994) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
