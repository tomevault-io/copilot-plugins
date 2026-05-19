## obsidian

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

Obsidian ナレッジベース（個人の学習ノート集）。コードプロジェクトではなく、Markdown ファイルの集合体。ビルド・テスト・リントは存在しない。

## 基本方針

- **常体**で記述する（敬語は使わない）
- **絵文字は使用しない**
- **図解を含める**（必要に応じて）
- **具体例を含める**（理解を助けるため）

## 1. フロントマターを必ず付ける

新規 `.md` ファイル作成時、先頭に YAML を置く。

```yaml
---
base: "[[ディレクトリ名.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - [カテゴリ名]
作成日時: YYYY-MM-DDTHH:mm:ss
aliases: [エイリアス1, エイリアス2]
---
```

### baseプロパティのルール

`base`プロパティは、そのノートが属するディレクトリ名（数字プレフィックスを除く）に設定する。

- ディレクトリ名から数字プレフィックス（`000_`, `010_`, `015_`, `020_`, `031_`, `050_`, `070_` など3桁）を除いた名前を使用
- 例：`000_Home/` → `[[Home.base]]`
- 例：`001_Task/` → `[[Task.base]]`
- 例：`002_Context/journal/` → `[[Context.journal.base]]` または `[[Context.base]]`
- 例：`003_Analysis/Weekly_Report/` → `[[Analysis.Weekly_Report.base]]` または `[[Analysis.base]]`
- 例：`010_CS/04_Network/` → `[[CS.Network.base]]` または `[[CS.base]]`
- 例：`015_Web/Browser/` → `[[Web.Browser.base]]` または `[[Web.base]]`
- 例：`020_AWS/` → `[[AWS.base]]`
- 例：`030_ECMAScript/` → `[[ECMAScript.base]]`
- 例：`031_Browser_runtime/` → `[[Browser_runtime.base]]`
- 例：`032_Node.js/` → `[[Node.js.base]]`
- 例：`033_npm/` → `[[npm.base]]`
- 例：`040_Vue.js/` → `[[Vue.js.base]]`
- 例：`050_Development_Environment/` → `[[Development_Environment.base]]`
- 例：`060_Claude_code/` → `[[Claude_code.base]]`
- 例：`070_Git/` → `[[Git.base]]`
- 例：`200_Life/Psychology/` → `[[Life.Psychology.base]]`
- 例：`200_Life/Health/` → `[[Life.Health.base]]`
- 例：`200_Life/Ideas/` → `[[Life.Ideas.base]]`

この設定により、ObsidianのDataviewプラグインでディレクトリごとにノートを分類・集約できる。

### エイリアス（aliases）のルール

ObsidianのAutomatic linkerプラグインは完全一致のみ対応のため、以下のルールで設定：

1. **タイトル名そのまま**を含める
2. **主要キーワードを分解**して含める（複合語の場合）
3. **英語の場合は大文字小文字のバリエーション**を含める（例：OAuth/oauth, REST/rest）
4. **スペースの有無のバリエーション**を含める（例：v-for 数字/v-for数字）
5. **技術用語の別表記**を含める（例：v-for/vfor/v for, v-on/von/v on）
6. **関連する概念や別名**も含める（例：setup構文に`<script setup>`, `script setup`）
7. **日本語と英語の両方**を含める（該当する場合）
8. **バージョンや別表記**を含める（例：OAuth2, OAuth 2.0）

例：
- `巻き上げとTDZ.md` → `[巻き上げとTDZ, 巻き上げ, TDZ, 巻き上げとtdz, 巻き上げとTdz]`
- `OAuth.md` → `[OAuth, oauth, OAuth2, OAuth 2.0]`
- `v-for 配列.md` → `[v-for 配列, v-for, vfor, v for, key]`

## 2. 本文の約束

- **常体**で書く（敬語なし）。**絵文字は使わない**
- セクション区切りは `---`
- 見出しや重要語は Obsidian の `[[リンク名]]` でリンクする
- 必要に応じて図解・具体例を入れる

## 3. ディレクトリ構造（3桁番号）

- `000_Home/`: ホーム
- `001_Task/`: プライベートタスク管理。**3者兼用プラットフォーム**として運用:
  - **OpenClaw 秘書ミア**（M4 Mac mini に固定常駐、`~/.openclaw/workspace`）: cron / heartbeat / 音声ジャーナル取り込み / Discord DM 通知などのプッシュ型・自律行動を担う
  - **本リポジトリの Claude Code（あなた）**: 複数の PC で動く可能性がある。ターミナルでユーザーと対話している間、起票・更新・整合性チェックを担うプル型。**個別タスクの `status` を書き換える時は必ず `Board.md` のカード位置も対応するカラムへ動かす責務**を負う（B-1b 同期。逆方向 UI ドラッグ → FM は Kanban Status Updater プラグインが自動。詳細は `001_Task/README.md`）。Mac mini で動いている時だけミアの workspace（`~/.openclaw/workspace/`）に直接アクセスできる。他 PC では見えない
  - **本人**: 喋る / 確認する / 必要なら直接編集
  - 1タスク = 1ファイル（`<slug>.md`）+ Kanban ボード（`Board.md`）+ Dataview ダッシュボード（`Dashboard.md`）の構造。仕様の Source of Truth は `001_Task/README.md`（frontmatter スキーマ・カラム ↔ status 値マッピング・cron 動作などは README.md を参照、CLAUDE.md 側では重複させない）
  - vault の cross-device 同期は git + remotely-save が担う。Mac mini 以外の PC で Claude Code を動かす時は、編集後に必ず remotely-save 同期を発火（`obsidian://advanced-uri?vault=obsidian&commandid=remotely-save:start-sync` を `open` で叩く）してから席を立つこと。ミアは Mac mini の vault しか見ないため、同期しないと反映されない
  - 衝突可能性は **Mac mini で Claude Code が動いている時だけ**（ミアと同 FS で並走）。「ほぼ発生しない前提で受け入れる」方針（特別なロック等は入れない）。他 PC からは git/remotely-save 経由で eventual consistency なので衝突しない（タイムラグはある）。これまでは `000_Home/Task.md` の Kanban プラグイン1枚で運用していたが移行中
- `002_Context/`: AI（Claude / OpenClaw）に渡す**コンテキスト原料**を置く場所。**加工前の生データのみ**。AI が解釈・要約・推論を加えたものは置かない（それは `003_Analysis/`）。データは2形式に分類:
  - **定量・時間駆動**: センサー / 自動収集で取れる時系列データ（部屋温度・睡眠時間・体温など）。サブフォルダは将来追加
  - **定性・自然言語**: 定量で拾えない感情・状態をカツキ自身が自然言語で記録するもの → `journal/`
  
  サブフォルダ：
  - **journal**: 定性・自然言語の入力先。カツキの感情・状態など定量センサーで拾えない要素を、自然言語で記録する場所。`YYYY-MM-DD.md` で蓄積。音声入力が主で、文体は話し言葉・句読点なし・誤変換そのままの音声認識生データ。整形済みノートではなく、後で秘書が解析してタスク化・週次振り返り・心理的傾向抽出などに使う**原材料**。ここのファイルは整形しない
- `003_Analysis/`: AI（Claude / OpenClaw）が解析・要約・推論した**加工後のアウトプット**を置く場所。`002_Context/` と対になる概念で、信頼境界が明確に違う（こちらは AI の解釈が入っている）。サブフォルダ：
  - **Weekly_Report**: 週次レビューを `YYYY-MM-DD.md`（その週の日曜の日付）で蓄積。OpenClaw が毎週日曜 21:00 に cron で生成する、過去7日のタスク完了状況と journal 要約をまとめたレポート
  - 将来的に `Monthly_Report/` / `Daily_Brief_Archive/` / `Trend/` などを追加予定
- `010_CS/`: コンピュータサイエンス系。サブフォルダ（番号順は低レイヤーから高レイヤー）：
  - **01_Computation_Theory**: 計算論、オートマトン、計算量理論、形式言語
  - **02_Hardware**: CPU、メモリ階層、ハードウェアアーキテクチャ
  - **03_OS**: オペレーティングシステム、プロセス、スレッド、仮想メモリ、スケジューリング、並行・並列
  - **04_Network**: ネットワーク層・トランスポート層（レイヤー4まで）、TCP/IP、UDP、ルーティング、DNS
  - **05_Database**: 永続化、ストレージ、トランザクション、Session/Cookieの「状態をどこに持つか」など
  - **06_Security**: 認証・認可、暗号理論、OAuth、境界、攻撃・防御など
  - **07_Programming_Languages**: 言語の理論、パラダイム、型、コンパイル・インタプリタ（具体言語は030_ECMAScript等で）
  - **08_Algorithm**: アルゴリズム、データ構造、計算量、オーダー
  - **09_AI**: 機械学習、ディープラーニング、NLP、コンピュータビジョン、強化学習
  - **10_Software_Engineering**: 設計・設計パターン、テスト、ビルド・CI、開発手法
- `015_Web/`: Web技術。サブフォルダ：
  - **HTTP**: HTTP、HTTPS、HTTP/2、HTTP/3、URI/URL
  - **WebSocket**: WebSocketプロトコル、双方向通信
  - **REST**: RESTアーキテクチャスタイル、REST API設計
  - **API**: API設計パターン、GraphQL、バージョニング、OpenAPI/Swagger
- `020_AWS/`: AWS関連
- `030_ECMAScript/`: ECMAScript（JavaScript言語仕様）関連。Vite等のフロント向けツール系ノートも暫定的にここに含む
- `031_Browser_runtime/`: ブラウザランタイム関連（JavaScriptエンジン、レンダリングエンジン、DOM、Web API（Fetch/Storage 等））
- `032_Node.js/`: Node.js実行環境（fs/path/process/Buffer/EventEmitter、CommonJS/ESM、標準モジュール、ランタイム挙動）
- `033_npm/`: npm関連（package.json、依存解決、コマンド、レジストリ、関連ツール）
- `040_Vue.js/`: Vue.js関連
- `050_Development_Environment/`: 開発環境・ツール（ESLint/Prettier、Docker、再現性など）
- `060_Claude_code/`: Claude Code 自体に関する学習ノート（CLAUDE.md の仕様、Auto Memory、MCP、トラブルシュートなど）。他の技術カテゴリと同列だが、**Claude Code がこの vault を読み書きする側でもある**点で特殊
- `070_Git/`: Git関連（基本操作、ブランチ運用、履歴操作、トラブルシュート）
- `200_Life/`: ライフ系。サブフォルダ：**Psychology**（心理学・UX）、**Health**（健康）、**Ideas**（アイデアや思考の整理）

## チェックリスト（新規作成時）

- [ ] 先頭に上記形式の YAML フロントマターがある
- [ ] `base` が配置ディレクトリに対応している
- [ ] `aliases` にタイトル・検索されそうな表記を含めている
- [ ] 常体・絵文字なし・セクションは `---`・リンクは `[[...]]`
- [ ] 適切な 3桁フォルダ・サブフォルダに保存している

## Obsidian プラグイン

- **automatic-linker**: エイリアス完全一致で自動リンク生成
- **remotely-save**: リモート同期
- **obsidian-kanban**: カンバンボード
- **obsidian-importer**: インポーター

## 注意事項

- 既存のファイル構造とスタイルを尊重
- すべての応答は日本語で行う
- ユーザーの指示に対して疑問や不明点がある場合は、必ず確認を取る
- ただし、明らかな誤変換やタイポ（例：「寿命と廊下」→「寿命と老化」）は、確認せずに適切に修正する

---
> Source: [katsuki-ishii/obsidian](https://github.com/katsuki-ishii/obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
