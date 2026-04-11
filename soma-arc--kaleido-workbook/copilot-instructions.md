## kaleido-workbook

> 本プロジェクトは PoC として「Poincaré 円板: 三鏡型 (p,q,r)／ジオデシック／角度スナップ／群展開／共役操作」を最小構成で検証します。役割詳細は docs/ROADMAP.md と各 Epic を参照し、本書ではルールの核のみを定義します。

# AGENTS（ルールの核）

本プロジェクトは PoC として「Poincaré 円板: 三鏡型 (p,q,r)／ジオデシック／角度スナップ／群展開／共役操作」を最小構成で検証します。役割詳細は docs/ROADMAP.md と各 Epic を参照し、本書ではルールの核のみを定義します。

- タイムゾーン: Asia/Tokyo
- 原則: シンプル・小さく作り速く回す（3分原則）
- ツール: TypeScript + React + Vite + pnpm / Canvas 2D / Biome(4スペース) / Vitest(coverage v8) / GitHub Actions

## 作業範囲（Scope）
- 読み取り専用: `tests/acceptance/**`（仕様ロック／人間が作成）
- 触らない: ビルド/CI 設定やワークフロー変更（別PRで合意のうえ実施）

## TDD ルール
- Red → Green → Refactor を徹底。
- 受け入れテストはロック。AI/エージェントは変更しない。
- プロパティテスト: fast-check（@fast-check/vitest）
  - 性質: 両円を満たす、回転/並進/一様スケール不変、入力順対称
  - 既定: `seed=424242`, `numRuns=200`（`vitest.setup.ts`。環境変数で上書き可）
- 近接数値: 比較は `toBeCloseTo(..., 12)` を基準（許容誤差はテスト側で定義）

## 禁止事項
- `tests/acceptance/**` の変更・削除・緩和
- 許容誤差や判定条件の無説明な改変
- Green のための恣意的なテスト改変

## DoD（Definition of Done）
- 受け入れ・ユニット・プロパティが全て Green（coverage v8）
- API シグネチャ/返却規約（順序・分類）が README と一致
- PR が対象 Issue を `Closes #<number>` で関連付け、GitHub Project 上の該当アイテムが `Done` に遷移

## コミット/PR ポリシー（最小）
- １作業単位で細かくコミットする
- pre-commit: `pnpm lint` / `pnpm format` / `pnpm test` を実行し失敗時ブロック
- pre-push: 少なくとも `pnpm test`
- commit message: Conventional Commits (英語)
- CI: typecheck / biome(lint/format-check) / test(coverage v8)

## コーディング規約
- 主要な公開 API（関数・クラス・型エイリアスなど）には TSDoc を付与すること

## 参照
- docs/ROADMAP.md（Now/Next/Later の方向性）
- README.md（クイックスタート/コマンド）
- GitHub Project（スプリントの Now/Next/Later のSSOT）/ Milestones（リリース管理）

メタ: 本書は TDD 方針へのピボット（2025-09-04）を反映。
メタ: Serena メモリを最新ルールに同期（2025-09-07）。


タスクに応じてリポ内の次のSSOTを読んで遵守してください:
ops/ai/prompts/v1/implement_issue.md # 実装フローの契約
ops/ai/playbooks/v1/dev_flow.md # Storybook用スタイル/DoD/テスト規約

実行:
ISSUE_NUMBERを基準に、依存が無いことを確認 → ブランチ {BRANCH_PREFIX}/{ISSUE_NUMBER}-{slug} を作成
storybook.md の規約に従ってコンポーネントのCSF3 stories/Controls/Docs/アクセシビリティ/Playテストを整備
小さくコミット（Conventional Commits、末尾に Refs #<id>）、lint/typecheck/test緑化
PULL_REQUEST_TEMPLATEにしたがってPR作成（テンプレを埋め、本文先頭に Closes #<id>）。
残件があれば新Issue提案（Epic/Parentに紐付け）
出力:
使ったSSOTのパスとバージョン（先頭行のv表記）を明記
作成ブランチ名／主要コミット件名／PR URL／残件の新Issue提案（タイトル＋1行）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/soma-arc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/soma-arc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
