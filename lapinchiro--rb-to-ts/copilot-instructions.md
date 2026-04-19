## rb-to-ts

> Ruby → TypeScript 変換トランスパイラ CLI ツール。

# CLAUDE.md

Ruby → TypeScript 変換トランスパイラ CLI ツール。

## Tech Stack

- **実装言語**: TypeScript (Node.js)
- **Ruby パーサー**: Prism Wasm 版
- **TS 生成**: ts-morph (軽量モード)
- **テスト**: vitest + スナップショット
- **Lint**: oxlint + eslint (7 層型安全構成)
- **フォーマット**: prettier
- **型チェック**: `tsc --noEmit`
- **未使用コード検出**: knip
- **パッケージ管理**: npm

## Key Commands

```bash
npm run build              # TypeScript compilation
npm run dev                # tsx で直接実行
npm run lint               # oxlint + eslint
npm run type-check         # tsc --noEmit
npm run format:check       # prettier check
npm run knip               # unused exports check
npm run test               # vitest run
npm run test:coverage      # vitest run --coverage
npm run quality            # type-check + lint + knip + test
```

## Architecture

変換パイプライン: Ruby source → `parser` (Prism AST) → `transformer` (IR) → `generator` (TS source)

```
src/
├── parser/            # Prism Wasm 呼び出し、Ruby AST 型定義
├── transformer/       # Ruby AST → TS IR 変換ロジック
├── generator/         # TS IR → TypeScript 出力 (ts-morph 軽量モード)
├── ir/                # 中間表現の型定義
├── types/             # 共有型定義
└── index.ts           # CLI エントリポイント
tests/
├── fixtures/          # 入力 Ruby + 期待出力 TS のペア
├── parser/            # パーサー単体テスト
├── transformer/       # 変換ロジックテスト
├── generator/         # 生成テスト
└── e2e/               # E2E テスト (Ruby 実行 == TS 実行の一致確認)
```

### IR 設計方針

IR は TypeScript AST 寄りに設計する。Transformer が Ruby 固有の意味解釈を担い、Generator は IR から TypeScript コードへの機械的な出力に徹する。

### ts-morph 軽量モード

生成フェーズでは軽量モード設定を使用し、型検証は後段の `tsc --noEmit` で行う。この「生成と検証の分離」を変更しない。

### エラーハンドリング

未対応の Ruby 構文に遭遇した場合:

1. 変換を中断しない — 変換可能な部分はすべて変換する
2. TS コメントとして埋め込む — `// [rb_to_ts] unsupported: <node_type> (line <N>)`
3. レポートを収集する — 未対応ノードの一覧を JSON 形式で標準エラー出力に出力する
4. 終了コードで区別する — 全変換成功: 0、一部未対応あり: 0 (警告出力)、パースエラー: 1

## Core Principles

以下の原則を常に遵守すること:

- **理想的な実装**: 開発コストを度外視し、論理的に最も理想的な実装を目指す。妥協やアドホックな対応を避け、型システム・アーキテクチャとして一貫した解法を選ぶ
- **KISS**: 過剰設計を避けよ。最小限の複雑さで現在の要件を満たすこと。ただし「理想的な実装」と矛盾する場合は理想を優先する
- **YAGNI**: 要求されていない機能・改善・拡張を作るな。今必要なものだけを実装せよ
- **DRY + 直交性**: DRY は「知識の重複」を排除する原則であり、「コードの見た目の重複」を排除する原則ではない。共通化によってモジュール間の結合が増えるなら、重複を残せ

## Code Conventions

- `any` 型の使用禁止 — 適切な型を定義すること
- 非 null assertion (`!`) の使用禁止 — 適切な null チェックを行うこと
- ES Modules (`"type": "module"`)
- ESLint 7 層型安全構成を適用 (詳細は `eslint.config.js` 参照)

## Quality Standards

全ての変更に対し **0 エラー・0 警告** を維持すること。作業完了時は /quality-check を実行する。

カバレッジ閾値のラチェット運用: 実測値が閾値を 2pt 以上上回ったら、閾値を 1pt 引き上げる。

## Testing

テストは **unit** と **integration** の 2 種類:

| 種類 | 配置 | 用途 |
|------|------|------|
| **unit** | `src/**/*.test.ts` | パーサー、変換ロジック、生成ロジックの個別テスト |
| **integration** | `tests/**/*.test.ts` | パイプライン全体の統合テスト |
| **スナップショット** | `tests/fixtures/` | 入力 Ruby → 変換 → スナップショット比較 |
| **E2E** | `tests/e2e/` | Ruby 実行 == TS 実行の一致確認 |

- テスト名: `describe('<対象>') > it('<条件>のとき<期待結果>')`
- テスト間の状態共有禁止

## 行動規範

- **変換可能性の独断禁止**: Ruby で表現されている情報や挙動を TypeScript にトランスパイルできない、という判断を独断で下さない。方法が見つからない場合はユーザーにヒヤリングする。「TypeScript に直接対応する構文がない」は設計課題であって不可能の証明ではない
- **判断軸のある質問**: 選択肢・メリデメ・推奨案を提示して確認する。「これでよいですか？」のような判断軸のない質問をしない。自分で判断できることは判断して進める
- **検証の原則**: 検証は「検証項目の列挙 → 期待結果の定義 → 実行 → 判定」の順。結果を見てから「期待通り」と後付けしない
- **デバッグ**: 修正が 1 回で成功しなかったら、次の修正前に根本原因の仮説を立てる。同じ修正を 2 回繰り返さない
- **後回しの記録**: スコープ外の課題を発見したら、内容と理由を `TODO` に詳細に記録する
- **ドキュメント同期**: コード変更時に plan.md, README.md, CLAUDE.md, doc コメントが不正確になっていないか確認・更新する

## ワークフロー

以下の状況では対応する Skill を必ず呼び出すこと:

- 新機能・バグ修正の着手 → /tdd
- 作業完了時 (コミット前) → /quality-check
- 機能追加完了後 → /refactoring-check
- **PRD (backlog/ のタスク) 完了後** → /backlog-management (TODO 更新 → backlog 削除 → plan.md 整理 → 次の PRD 着手の順序を厳守)
- 開発セッションの最後 (コミット前) → /todo-audit
- backlog/ の操作 → /backlog-management
- backlog/ が空で作業依頼を受けた → /backlog-replenishment
- PRD の作成 → /prd-template
- TODO が空で作業依頼を受けた → /todo-replenishment
- 調査タスク → /investigation
- TODO の棚卸し (定期的、または大きな機能追加の完了後) → /todo-grooming
- abbrev 変換改善の開発ループ → /abbrev-cycle
- ルールの作成・変更 → /rule-writing, /rule-maintenance

## 自発的改善の原則

問題や不整合を発見したら、ユーザーに指摘される前に自発的に調査・修正すること:

- 警告・エラー・不整合を「一時的な問題」として安易に無視しない
- 問題の根本原因を特定してから対処する
- 「動いているから良い」ではなく「正しい状態か」を基準にする

## スキルの自己改善

### 振り返り (Observe)

スキル実行中に以下のいずれかに気づいたら、実行完了後に `TODO` へ `[skill-feedback:<スキル名>]` タグ付きで記録する:

- スキルの指示が曖昧で、判断に迷った箇所があった
- スキルのステップが現在のコードベースや環境に合わなくなっていた
- ユーザーがスキルの途中で方向修正を求めた
- スキルに書かれていない判断を自分で補った

### 能動的改善 (Amend)

スキル実行中に改善点に気づいた場合、実行完了後にユーザーに改善提案を行う。

### 受動的学習

ユーザーから Claude 自身の挙動に関する修正指示を受けたとき:

1. 指示を一般化・抽象化する
2. 保存先を判断する:
   - プロジェクト固有のルール → `.claude/rules/` に追記または新規作成
   - プロジェクト固有の手続き → `.claude/skills/<name>/SKILL.md` に作成
   - 個人的な好み → `~/.claude/CLAUDE.md` に追記
3. 書き込む内容と保存先をユーザーに提示し、確認を得てから書き込む

## Gotchas

- 静的解析の出力確認は、必ずファイルにリダイレクトしてから Read ツールで確認する。`tail` や `grep` でのフィルタリングは出力が切れるリスクがあるため禁止

  ```bash
  npm run lint > /tmp/lint-result.txt 2>&1
  npx tsc --noEmit > /tmp/typecheck-result.txt 2>&1
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lapinChiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
