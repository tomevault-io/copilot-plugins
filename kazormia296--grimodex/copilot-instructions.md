## grimodex

> Electron + React 19 + TypeScript。TipTapベースのリッチテキストエディタに

# Grimodex — AI統合小説執筆エディタ

## プロジェクト概要

Electron + React 19 + TypeScript。TipTapベースのリッチテキストエディタに
AIチャットパネルとCodex/Snippet抽出機能を組み合わせた小説執筆ツール。
**コア体験: AIとのチャットから知識を抽出し、構造化して執筆に活かす。**

現行デスクトップランタイムは Electron。renderer は typed preload IPC を介して
main process を呼び、Rust 実装は N-API モジュールと standalone MCP から共有する。
`src-tauri` 直下の Tauri シェルは v1 互換・移行確認用に凍結した legacy コードであり、
新機能の実装先にしない。`src-tauri/crates/` の共有 crates と MCP は引き続き現役。

## コマンド

- デスクトップ開発: pnpm electron:dev
- フロントのみ: pnpm dev
- ビルド: pnpm electron:build
- パッケージ: pnpm electron:package
- テスト: pnpm test
- テスト(単体): pnpm test --run [ファイルパス]
- フロントCI相当: pnpm verify:frontend
- Electronテスト: pnpm test:electron --run
- N-APIビルド: pnpm napi:build
- N-APIテスト（上記ビルド後）: pnpm --dir electron/native/grimodex-node test
- Lint: pnpm lint:fix
- 型チェック: npx tsc --noEmit
- Electron型チェック: pnpm exec tsc -p electron/tsconfig.json --noEmit
- N-APIチェック: cargo check --manifest-path electron/native/grimodex-node/Cargo.toml
- 共有Rustチェック: cargo check --manifest-path src-tauri/Cargo.toml --workspace --exclude grimodex --features grimodex-semantic/semantic-embedding
- 共有Rustテスト: cargo test --manifest-path src-tauri/Cargo.toml --workspace --exclude grimodex --features grimodex-semantic/semantic-embedding

## GitHub認証（Codex sandbox）

- 通常のユーザー端末では `gh auth status` が成功していても、Codex の sandbox 内では
  OS keyring を参照できず、`The token in default is invalid` と誤判定されることがある。
- sandbox 内の失敗だけを根拠に、ユーザーへ再ログインを依頼したり `gh auth logout` を
  実行したりしない。まず同じ `gh auth status` を `require_escalated` で再実行し、
  sandbox 外の keyring から認証状態を確認する。
- 認証が sandbox 外で成功した場合、keyring／ネットワークを必要とする `gh`・`git`
  操作も、必要な範囲に限定して escalation して続行する。トークン本体は出力しない。
- sandbox 外でも失敗した場合に限り、`gh auth login -h github.com` をユーザーへ案内する。

## コード規約

- ES modules（import/export）、CommonJS禁止
- 2スペースインデント、TypeScript strictモード
- React: 関数コンポーネント + hooks のみ
- 状態管理: グローバル=Zustand、局所=Jotai
- DB操作: Drizzle ORM経由、生SQL禁止
- テスト: Vitest、ソースと同階層に \*.test.ts
- コンポーネント: 1ファイル1コンポーネント、200行超えたら分割
- Rust: unwrap()禁止、thiserror/anyhow使用（共有 crates の詳細は src-tauri-CLAUDE.md）
- アニメ: duration/easing は `src/lib/animation.ts` の `DURATIONS`/`EASINGS`/`VARIANTS` 経由（べた書き禁止、詳細は /polish-motion）
- React/TS詳細は src-CLAUDE.md を参照

## アーキテクチャ原則

- feature-based ディレクトリ構造（src/features/[name]/）
- renderer は Node/Electron/N-API を直接 import せず、`window.grimodex` の typed preload API を使う
- IPC の正本は `electron/shared/ipcContract.ts`。main 側で引数を再検証し、N-API backend を呼ぶ
- Rust のドメイン実装は `src-tauri/crates/` に置き、Electron 用 binding は `electron/native/grimodex-node/` に置く
- 既存経路の責務・型・契約・データフローを renderer／Electron IPC／N-API／shared Rust／MCP、
  または複数 AI provider 経路のうち2つ以上で協調変更する場合は、`/refactor-cross-boundaries` を使用し、実装前に影響マトリクスを作る
- AI指示、system prompt、Codex skill、AI policy、AI経路レジストリ、評価fixtureなどの
  AI behavior asset を作成・変更する場合は `/grimodex-author` を使用し、実在する正本と評価証跡を一緒に更新する
- AI behavior asset の変更後、および commit／PR 前の差分評価では `/grimodex-impact-gate` を使用し、
  canonical quality workflow が選択した Light suite を読み取り専用で実行する
- 新機能は `/implement-feature`、新規 Electron IPC command は `/add-electron-command` を優先する
- バグ修正は `/debug-issue` を主フローとし、上記の横断条件を満たす場合は影響マトリクスも併用する
- Electron release CI／release workflow／tag build／署名・公証／installer migration／artifact publish
  の失敗は `/debug-release-ci` を主フローとする。一般のPR／master CI失敗は `/debug-issue` を使う
- release CI修正中はversion、release notes、tagを変更せず、失敗stage相当のfocused gateを先に通す。
  workflow再実行だけを目的にpatch versionを上げない。修正merge後のDraft Release準備だけを `/bump-version` へ渡す
- Grimodexの公開向け文書は `docs/communication-style-guide.md` を正本として `/write-grimodex-copy`
  を使用する。Issue、PR、コミット、UI／エラー／復旧手順、API／IPC／MCP／CLI／セットアップ等の
  技術文書には適用しない
- version更新を含むリリース作業は `/bump-version` を主フローとし、リリースノート作成だけを
  `/write-grimodex-copy` に委ねる。release commit後のpush／PR／mergeは `/ship-branch` へ渡す。
  tag workflow完了後もGitHub ReleaseはDraftのまま停止し、公開には別の明示指示を必要とする
- SQLite WALモード、FTS5有効
- エディタ: チャプター/シーンごとに独立TipTapインスタンス
- AIチャット: シーンごとに独立した会話履歴を保持
- 帰属追跡: テキスト挿入時にsource metadata（human/ai/unknown）を記録

## スキル発火条件

| トリガーワード                                                                                              | 発動スキル                 | 動作                                   |
| ----------------------------------------------------------------------------------------------------------- | -------------------------- | -------------------------------------- |
| 「調べて」「調査」                                                                                          | /explore-codebase          | コード探索                             |
| 「AI指示」「システムプロンプト」「Codexスキル」「AI評価fixture」                                            | /grimodex-author           | 正本→評価証跡→差分品質ゲート           |
| 「差分評価」「品質ゲート」「impact gate」「コミット前／PR前評価」                                           | /grimodex-impact-gate      | 差分→関連Light suite選択・実行         |
| 「実装して」「作って」                                                                                      | /implement-feature         | 実装フロー                             |
| 「レビュー」                                                                                                | /review-code               | コードレビュー                         |
| 「テスト」                                                                                                  | /test-feature              | テスト作成・実行                       |
| 「release CI失敗」「release workflow失敗」「tag build失敗」「packaging失敗」「署名」「公証」「publish失敗」 | /debug-release-ci          | run／SHA固定→対象stageだけ検証         |
| 「デバッグ」「修正」「一般CI失敗」                                                                          | /debug-issue               | 一般デバッグフロー                     |
| 「Electronコマンド」「IPC」「invoke」                                                                       | /add-electron-command      | IPC一括追加                            |
| 「アニメ」「トランジション」「動き」「磨いて」                                                              | /polish-motion             | UIモーション規律                       |
| 「横断リファクタ」「境界整理」「責務移動」「パイプライン再編」                                              | /refactor-cross-boundaries | 影響マトリクス→段階レビュー→CI相当検証 |
| 「リリースノート」「広報文」「告知文」「README冒頭」「日英リリースノート」                                  | /write-grimodex-copy       | 正本確認→事実整理→日英整合→公開前確認  |
| 「バージョン上げて」「リリースタグ」「リリース準備」                                                        | /bump-version              | version→文面→PR→tag→Draft確認          |
| 「PR出して」「プルリク作って」「pushしてマージ」「shipして」                                                | /ship-branch               | push→PR→CI・レビュー→squash merge      |

## Cursor Cloud specific instructions

Cursor Cloud VM 固有の非自明な注意点だけを記す。コマンド一覧は上の「コマンド」節が正本。
起動時の update script は `pnpm install --frozen-lockfile` と Electron バイナリ取得のみを行う。

- **sudo は使用不可**（cloud harness がブロックする）。Electron 実行 / N-API ビルドに必要な
  システムライブラリ（`libgtk-3-0t64` / `libnss3` / `libgbm1` / `libsecret-1-dev` / build-essential 等）は
  ベースイメージに導入済みなので、通常は追加の apt インストールは不要。新しい system パッケージが
  どうしても要る場合は、勝手に回避策を探さずユーザーへ依頼する。
- **Rust の C++ リンク（重要・非自明）**: 新しめの stable Rust は x86_64-unknown-linux-gnu で
  self-contained な `rust-lld` を既定リンカにするが、これは gcc 私有ディレクトリを検索しないため、
  C++ を引く crate（tokenizers / ort / lindera＝`grimodex-node` や共有 Rust のテストバイナリ）が
  `unable to find library -lstdc++` で失敗する。マシン全体の cargo 設定
  `/usr/local/cargo/config.toml`（CARGO_HOME）に `-L /usr/lib/gcc/x86_64-linux-gnu/13` を追加して
  回避済み。この 1 行が消えると `pnpm napi:build` と `cargo test/clippy --all-targets` が壊れる。
- **N-API ネイティブモジュール**: `electron/native/grimodex-node/grimodex-node.node`（約260MB）は
  snapshot に prebuild 済み。`src-tauri/crates/*` か N-API crate（`electron/native/grimodex-node`）を
  変更したときだけ `pnpm napi:build` で再ビルドする（release ビルドで約3〜5分）。未ビルドでも
  Electron は fail-soft 起動し、DB/AI 系 IPC が `IPC_BACKEND_UNAVAILABLE` を返すだけになる。
- **Electron バイナリ**: `.npmrc` に `ignore-scripts=true` があるため `pnpm install` では
  Electron 本体バイナリがダウンロードされない。`node node_modules/electron/install.js`（冪等）で取得する。
  update script がこれを実行するので通常は手動不要。
- **アプリの起動（GUI）**: `DISPLAY=:1` の Xvfb 上で動く。コンテナに Chromium サンドボックスが無いため
  `ELECTRON_DISABLE_SANDBOX=1 pnpm electron:dev` で起動すること。起動ログに出る
  `Failed to connect to the bus`（dbus）系エラーは system D-Bus 不在によるもので無害。
  `pnpm electron:dev` は Vite(1430) + esbuild watch + Electron を一括起動する。初回は利用規約同意
  ダイアログ→ランチャー→「新規プロジェクト...」の順で入る。

---
> Source: [kazormia296/Grimodex](https://github.com/kazormia296/Grimodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
