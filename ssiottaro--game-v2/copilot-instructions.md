## game-v2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 現状

**設計フェーズ — コードはまだ存在しない。** リポジトリには設計ドキュメントのみが置かれている。最初の実装スプリント（Sprint 1）は空のプロジェクトから Phaser/Vite を立ち上げるところから始まる。プロジェクトがスキャフォールドされるまでビルド・Lint・テスト等のコマンドは存在しない。

実装開始を依頼されたら、公式テンプレートを使うこと：

```bash
pnpm create @phaserjs/game@latest
# 選択：TypeScript + Vite
```

パッケージマネージャは **pnpm** を使用する（`npm` / `yarn` は使わない）。

技術スタック（決定済み、`architecture-decision-record.md` 参照）：

- **TypeScript** + **Vite**
- **プラットフォームUIレイヤー**：素のHTML/CSS/TS（Title / GameSelect / Settings / Ranking / Result）。UIフレームワーク（React / Svelte / Vue）は採用しない
- **個別ゲーム本体**：**Phaser.js**（ゲームの `Game` シーンのみ）。`PhaserHost` が `<canvas>` を mount/unmount
- ホスティング：**Cloudflare Pages**（main への push で自動デプロイ）
- スコア保存：**Supabase**（PostgreSQL + REST）

## プロジェクトの根幹（スコープに触れる前に必ず理解すること）

これは**単一のゲームではなく、Webゲームプラットフォーム**である。オーナーが時間をかけて複数のゲームを追加していく構成で、すべての作品は二人のキャラクター — 紗倉るる（宇宙人キャラ）と ぷかるん（地球語通訳役の相棒）— の世界観を共有する。

- **オーナー単独運営モデル**：ゲームを追加するのはプロジェクトオーナーのみ。第三者からの投稿は受け付けないため、サンドボックス機構は不要。
- **ランキングはゲームごとに独立**：横断集計、総合ポイント、横串実績は作らない。
- **認証なし**：プレイヤー識別は `localStorage` に保持される自由入力の名前のみ。複数ゲームで同じ名前を使っていても「同一プレイヤー」として扱うのは慣習上のことで、システム的な強制はしない。
- **非営利のファン活動（推し活）**：比重は「愛でる：ゲーム性 = 7：3」。「より面白い」と「よりキャラらしい」が純粋に拮抗したら、キャラらしさを優先する。
- **公式イラストの直接利用は基本NG**：本素材が依頼で揃うまでは仮素材（フリー素材／AI生成／単純な図形）で進める。「公式イラストを使っている」と称してはいけない。

### キャラクター制約

- **紗倉るる** は喋る（日本語テキストのセリフ）。性格は「昼は元気、夜はおっとり」。シンボルカラーは緑（`#7FBCB2`、`#3F8E85`、アクセント `#E5A8B2`、クリーム `#F6F0DF`）。好きなもの：メロンソーダ、RADWIMPS、シュタインズゲート、キングダム、歌、クラシックギター。嫌いなもの：礼儀がない人。
- **ぷかるん** は言葉を発さない。鳴き声＋モーションのみ。これは「設定が定まっていないキャラの言動を勝手に作らない」という意図的な決定。**オーナーの承認なしに ぷかるん のセリフを追加してはいけない**。
- 両キャラの好物・設定はゲーム内のアイテムや小ネタの源泉として活用する（例：Game #1 のレアアイテムは メロンソーダ缶／桜の花びら／クラシックギター で、すべてキャラ設定に由来）。

## ドキュメントマップ

| ファイル | 役割 |
|----------|------|
| `architecture-decision-record.md` | 技術選定の根拠、Cloudflare Pages / Supabase の詳細 |
| `platform-design.md` | **プラットフォーム層の設計**：シーン構成、共通サービス、DBスキーマ、ゲーム追加ワークフロー |
| `games/001-pukarun-run.md` | Game #1（エンドレスラン）の仕様。今後追加するゲームの仕様書テンプレート |
| `game-concept-draft.md` | オーナーの回答入りヒアリングシート。元の意図を確認したいとき参照 |
| `ideas.md` | **構想メモ（ブレスト用）**：未確定の演出・小ネタ、Game #2以降の候補、プラットフォーム全体の世界観アイデア。育ったら各仕様書へ昇格 |

設計判断がプラットフォームか個別ゲームか曖昧なときは、**共通領域は `platform-design.md` を正とする**。個別ゲーム固有のルール・データは各ゲームのファイルが責任を持つ。

## アーキテクチャの規約

### ディレクトリ構成（コード作成後）

```
index.html       # SPA エントリ
src/
  main.ts        # ルーター起動
  platform/      # 共通シェル（HTML世界）
    router.ts    # ハッシュベースの簡易ルーター
    phaserHost.ts # ゲーム本体の Phaser インスタンスを mount/unmount
    views/       # HTMLビュー：title, gameSelect, settings, gamePlay, result, ranking
    services/    # SupabaseClient, ScoreService, PlayerNameStore, SoundManager, PendingResult
    ui/          # 共通HTMLパーツ（button, modal, toast）
    styles/      # 共通CSS（reset, tokens, layout）
    registry.ts  # GameSelect に表示するゲーム一覧
    ngwords.ts   # プレイヤー名用の簡易NGワードリスト
  games/
    pukarun-run/ # 1ゲーム = 1ディレクトリ。MiniGameDefinition を default export
                 # 中身は scenes/（Phaser）と assets/、config.ts のみ
```

### HTML世界とPhaser世界の境界

- **プラットフォーム画面（Title / GameSelect / Settings / Result / Ranking）は素のHTML/CSS/TS**で書く。Phaser シーンとして実装してはいけない
- **Phaser は個別ゲーム本体（`src/games/*/scenes/`）でのみ使用**。プラットフォーム側からは `PhaserHost.mount()` 経由で起動する
- ゲーム終了時、Phaser シーンは `this.game.events.emit('gameOver', { score, metadata })` を発火する規約。`PhaserHost` がそれを受けて HTML ルーターに通知し Result 画面へ遷移
- ビュー関数のシグネチャは `render(container, params?): () => void`（戻り値はクリーンアップ関数）

### ゲーム追加ワークフロー（このシンプルさを必ず維持する）

新しいゲームを追加する作業 = **`src/games/` 配下に1ディレクトリ作成 + `registry.ts` に `import` 1行追加**。この制約を壊す変更（ゲームごとのスキーマ追加、複数ファイルにハードコードされたゲーム一覧など）はアンチパターン。

### データベース

全ゲームは**1つの `scores` テーブル**を `game_id` で共有する。ゲーム固有のフィールドは `metadata` JSONB カラムに格納すること。**ゲームごとに専用カラムや専用テーブルを作ってはいけない**。RLS は anonymous の INSERT/SELECT を許可、UPDATE/DELETE は不可。スキーマは `platform-design.md` 第4節を参照。

### 明示的に作らないもの

オーナーがプラットフォーム本体としては作らないと決めた項目（将来「別のゲーム」としては登場し得るが、プラットフォーム機能としては入れない）：

- 認証 / ユーザーアカウント
- 横断ランキング、総合ポイント、横串実績
- X (Twitter) 向け結果画像シェア機能（MVP外、初回リリースには含めない）
- ぷかるん が喋るための後付け対応（喋らない、で確定）
- Game #1 内の育成・強化要素（別ゲームの候補としては残るが、Game #1 には入れない）

## 進め方の規範

- **スプリントは1週間**。スプリント計画は `games/001-pukarun-run.md` 第10節。各スプリント末に動くビルド + ファンヒアリング。
- **公開目標：2026年6月**（キャラの3周年）。スコープ削減は日付ずらしより優先。
- **公開方式**：URLをファンに共有する形。ログイン壁・パスワードゲートは設けない。
- ユーザー向けUIテキストおよび設計ドキュメントは日本語で書く。コードの識別子およびコードコメントは英語。

---
> Source: [SsioTTaro/game-v2](https://github.com/SsioTTaro/game-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
