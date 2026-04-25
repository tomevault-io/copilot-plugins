## kiri

> 本レポジトリの設計知は `docs/overview.md` を起点としたテーマ別ドキュメントに分割されています。TypeScript での実装を追加する際は `src/indexer/`（Git 走査と DuckDB 取り込み）、`src/server/`（MCP サーバー）、`src/client/`（CLI や Codex 用ユーティリティ）を基本フォルダとし、共通ロジックは `src/shared/` に集約してください。設定スキーマは `config/` に YAML で格納し、型定義を `types/` へ配置します。生成物や DuckDB のワークファイルは `var/` 以下にまとめ、テストは対象モジュールを鏡写しにした `tests/indexer/*.spec.ts` などへ置いてください。

# Repository Guidelines

## プロジェクト構成とモジュール整理

本レポジトリの設計知は `docs/overview.md` を起点としたテーマ別ドキュメントに分割されています。TypeScript での実装を追加する際は `src/indexer/`（Git 走査と DuckDB 取り込み）、`src/server/`（MCP サーバー）、`src/client/`（CLI や Codex 用ユーティリティ）を基本フォルダとし、共通ロジックは `src/shared/` に集約してください。設定スキーマは `config/` に YAML で格納し、型定義を `types/` へ配置します。生成物や DuckDB のワークファイルは `var/` 以下にまとめ、テストは対象モジュールを鏡写しにした `tests/indexer/*.spec.ts` などへ置いてください。

GitHubを利用している。IssueはGitHubで管理している。GitHub mcpでアクセスする。

GitHub: CAPHTECH/kiri

## ビルド・テスト・開発コマンド

Node.js 20 と pnpm を前提とします。主要コマンドは以下のとおりです。

- `pnpm install`: 依存関係を解決し、`node_modules` を整備します。
- `pnpm run build`: `tsc --noEmit false` で `dist/` に ESM を出力します。
- `pnpm run dev`: `tsx src/server/main.ts --port 8765` を起動しホットリロードを有効化します。
- `pnpm run lint`: `eslint "src/**/*.{ts,tsx}"` と `prettier --check` を実行します。
- `pnpm run test`: `vitest run --coverage` を呼び出し、閾値 `lines ≥ 0.8` を確認します。
- `pnpm run check`: `pnpm run lint` と `pnpm run test` をまとめて実行します。
  開発で固定化した診断コマンドは `scripts/diag.ts` に関数として追加し、`pnpm exec tsx` で再利用してください。

## コーディングスタイルと命名

TypeScript 5 系を採用し、インデントは 2 スペース、セミコロンは常に付与します。変数・関数は `camelCase`、クラスと型は `PascalCase`、定数は `SCREAMING_SNAKE_CASE` を用いてください。`tsconfig.json` の `strict` と `noUncheckedIndexedAccess` は必ず有効化し、`eslint-config-standard-with-typescript` と `prettier` を pre-commit で実行します。SQL 定義は `sql/` に `.sql` 拡張子で保存し、キーワードは大文字、識別子は `snake_case` に統一します。MCP ツール名は `<領域>.<動詞>`（例: `context_bundle`）の形式とし、例外メッセージは「原因→対処」を 2 文で記述してください。

## テスト指針

テストは `vitest` を標準とし、`tests/server/context_bundle.spec.ts` のように対象モジュールを明示する命名を徹底します。DuckDB を利用するテストは `tmp` ディレクトリにワークファイルを生成し、`afterEach` で削除してください。検索品質など長時間の評価は `tests/eval/*.spec.ts` にゴールデンデータを置き、P@10 と TTFU をメトリクスとして `vitest --runInBand` で収集します。Pull Request では新規機能に対して 80% 以上のステートメントカバレッジを確保してください。

## コミットとプルリクガイド

Conventional Commits を必須とし、例として `feat(server): expose semantic rerank` や `chore(deps): bump duckdb-wasm` の形式を用います。1 コミットで 1 論点を守り、生成物や `var/` 配下は含めないでください。Pull Request では「目的」「実装概要」「検証手順」「残課題」を見出しとして記載し、関連 Issue があれば `Closes #番号` で紐付けます。レビュー依頼前に `pnpm run check` の結果をログで共有し、性能差分がある場合はベンチ結果を表形式で添付してください。

## セキュリティと設定メモ

denylist（`secrets/**`, `*.pem`, `.env*` など）は `.gitignore` と indexer 内のフィルタで二重に適用してください。DuckDB ファイルは暗号化ディスク上の `var/index.duckdb` を既定にし、MCP 応答では機密パスを `***` へマスクします。外部へのコンテキスト共有時は `scripts/audit/export-log.ts` を経由して行範囲ログを残し、週次でレビューしてください。依存更新時は `pnpm up --latest` を `scripts/update-deps.ts` 経由で実行し、差分と署名確認結果を PR コメントで共有する方針を維持します。

## MCP サーバー運用メモ

- `pnpm run dev` や `scripts/assay/*` から `src/server/main.ts` を起動すると、ポート `19999` / `20099` のサーバープロセスが常駐します。**複数同時起動やプロセス放置は禁止**です。作業前後に `ps aux | grep 'src/server/main.ts'` を確認し、不要な Node プロセスは `kill` で停止してください。古いプロセスが残ると検索結果が更新されず、Assay も古い挙動を報告します。
- DuckDB を更新するジョブ（`pnpm exec tsx src/indexer/cli.ts ...` や `pnpm run assay:*`）を実行する際は、必ず上記のサーバープロセスを全て停止した状態で行います。起動中のままインデクサを走らせると `var/index.duckdb.lock` が残り、`Database was already closed` のまま復旧できなくなります。
- もし手動でロック解除が必要になった場合は、**プロセス停止 → `rm var/index.duckdb.lock` → `pnpm exec tsx src/indexer/cli.ts --full`** の順でのみ実施してください。ロック解除前にプロセスを殺さないとデータ破損につながります。
- `external/assay-kit` は private submodule です。`pnpm run eval:golden` や `scripts/assay/*` が同リポジトリ内のファイルをそのまま参照するため、評価ログや JSON を外部へ共有しないでください。成果物は `var/assay/` や `tmp/` など gitignore 配下に保存し、パスを開示する場合は `***` でマスクします。アダプター実行前に既存 `kiri` サーバーを必ず停止しないと古いポートに接続され、最新インデックスが eval に反映されません。

## 評価ベースライン運用

- リリース挙動との比較は `pnpm exec tsx scripts/assay/run-evaluation.ts --profile release`／`--profile current` で行います。スクリプトが `KIRI_SUPPRESS_NON_CODE` などのフラグを自動で切り替えるため、MCP サーバーを起動する前に必ず該当コマンド経由で評価してください。同日内の結果は `var/assay/eval-<profile>-YYYY-MM-DD.(json|md)` に自動保存されます。
- `npx kiri-mcp-server@0.10.0` を使ったベースデータ収集は `scripts/assay/collect-base-data.ts` から行い、成果物を `var/assay/base/` に置きます。外部配布禁止のため、ファイルパスを共有する際は `***` でマスクし、リポジトリ外へ raw JSON/ログを持ち出さないでください。
- `pnpm run eval:golden -- --verbose` は VS Code DB（`external/vscode/.kiri/index.duckdb`）を使用します。開始前に `ps aux | grep 'src/server/main.ts'` で既存サーバーを落とし、完了後に再度確認してください。評価ログ（`var/eval/latest.(json|md)`）は ignore 対象なので誤コミットに注意します。

## pce-memory とは

プロジェクトの知識（設計決定、規約、好み、タスク）を永続化し、セッションを跨いで想起するためのMCPサーバーです。

## 利用フロー

1. activate → タスク開始前に関連知識を想起
2. 作業実行 → 想起した知識を考慮して実装
3. upsert → 重要な決定事項を記録
4. feedback → 知識の有用性をフィードバック

## いつ activate するか

以下の状況で `pce.memory.activate` を呼び出してください：

- **新しいタスクを開始するとき** - 関連する過去の決定を確認
- **設計判断が必要なとき** - 既存のアーキテクチャ決定を参照
- **命名や規約を確認したいとき** - プロジェクト固有のルールを想起
- **エラーに遭遇したとき** - 過去の類似問題の解決策を検索

```json
// 例: API設計前に関連知識を想起
{
  "q": "API 命名規約 RESTful",
  "scope": ["project", "principle"],
  "allow": ["answer:task"],
  "top_k": 10
}
```

## いつ upsert するか

以下の情報を `pce.memory.upsert` で記録してください：

### kind: fact（事実）

- アーキテクチャ決定（「認証にはJWTを使用」）
- 技術的制約（「DuckDBはFOREIGN KEY CASCADEをサポートしない」）
- API仕様（「POST /cancel は非同期で202を返す」）

### kind: preference（好み）

- コーディングスタイル（「関数型パターンを好む」）
- ツール選択（「テストにはVitestを使用」）
- 命名規則（「ハンドラは handle\* プレフィックス」）

### kind: task（タスク）

- 進行中の作業（「認証機能を実装中」）
- TODO項目（「エラーハンドリングを追加する必要がある」）

### kind: policy_hint（ポリシーヒント）

- セキュリティ要件（「PII は internal 以上で保護」）
- 運用ルール（「本番DBへの直接アクセス禁止」）

```json
// 例: 設計決定を記録
{
  "text": "状態管理にはfp-tsのEitherを使用し、例外をthrowしない",
  "kind": "fact",
  "scope": "project",
  "boundary_class": "internal",
  "provenance": {
    "at": "2025-11-27T15:00:00Z",
    "actor": "claude",
    "note": "ADR-0002で決定"
  },
  "content_hash": "sha256:..."
}
```

## scope の使い分け

| scope       | 用途             | 例                                              |
| ----------- | ---------------- | ----------------------------------------------- |
| `session`   | 今回の会話限定   | 「このファイルを編集中」「デバッグ中の仮説」    |
| `project`   | プロジェクト固有 | 「JWTで認証」「Vitestでテスト」「REST API設計」 |
| `principle` | 普遍的原則       | 「TDDで開発」「SOLID原則を遵守」                |

## boundary_class の使い分け

| class      | 用途         | 例                                         |
| ---------- | ------------ | ------------------------------------------ |
| `public`   | 公開可能     | OSSライブラリの使い方、一般的な技術情報    |
| `internal` | 社内限定     | 内部API仕様、アーキテクチャ決定            |
| `pii`      | 個人情報含む | ユーザー名、メールアドレスを含む文脈       |
| `secret`   | 機密情報     | APIキー、認証情報（※記録しないことを推奨） |

## feedback を送るタイミング

activateで取得した知識が役立ったかを `pce.memory.feedback` で報告してください：

| signal      | いつ送るか                                |
| ----------- | ----------------------------------------- |
| `helpful`   | 提案/知識が採用された、問題解決に貢献した |
| `harmful`   | 提案が誤っていた、バグの原因になった      |
| `outdated`  | 情報が古くなっていた、現状と異なる        |
| `duplicate` | 同じ内容が既に別のClaimで存在する         |

```json
// 例: 知識が役立った
{
  "claim_id": "clm_abc123",
  "signal": "helpful"
}
```

## 実践的なワークフロー例

### 1. 新機能実装

```
User: "ユーザー認証機能を実装して"

Agent:
1. activate({ q: "認証 JWT セッション", scope: ["project"] })
   → 既存の認証関連の決定を確認

2. 実装を進める（想起した知識を考慮）

3. upsert({
     text: "認証トークンは15分で期限切れ、リフレッシュトークンは7日",
     kind: "fact",
     scope: "project"
   })
   → 新しい決定を記録

4. feedback({ claim_id: "clm_xxx", signal: "helpful" })
   → 参考になった知識にフィードバック
```

### 2. バグ修正

```
User: "ログインできないバグを直して"

Agent:
1. activate({ q: "ログイン 認証 エラー", scope: ["project", "session"] })
   → 過去の類似問題や関連実装を確認

2. 原因を特定して修正

3. upsert({
     text: "JWTのtimezone不一致でログイン失敗する問題があった。UTCに統一",
     kind: "fact",
     scope: "project"
   })
   → 解決策を記録（将来の参考に）
```

### 3. コードレビュー

```
Agent:
1. activate({ q: "コーディング規約 命名 スタイル", scope: ["project", "principle"] })
   → プロジェクトの規約を確認

2. レビュー実施（規約に基づいて）

3. 新しい規約が決まったら upsert
```

---

## 注意事項

- **secret は記録しない**: APIキー、パスワード等は upsert しないでください
- **content_hash は必須**: 重複防止のため、テキストのSHA256ハッシュを含めてください
- **provenance を明記**: いつ、誰が、なぜその知識を記録したか追跡可能にしてください
- **過度な記録は避ける**: 全ての会話を記録するのではなく、重要な決定のみを厳選してください

---
> Source: [CAPHTECH/kiri](https://github.com/CAPHTECH/kiri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
