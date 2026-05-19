## job-hunting-copilot

> このドキュメントは、このリポジトリで Codex を使う際の開発ルールと原則をまとめたものです。

# Codex ガイドライン

このドキュメントは、このリポジトリで Codex を使う際の開発ルールと原則をまとめたものです。
日本語で返答してください。

## 技術スタック

- Next.js (App Router) + TypeScript + React + Tailwind CSS v4
- Supabase (Auth/DB/Storage)
- AI: `lib/ai/` のラッパー経由で呼び出し

## 開発原則（必須）

- **TypeScript/React の原則を尊重**: 型安全・明確な責務分離・コンポーネントの単一責任を守る。
- **YAGNI**: 今必要な機能のみ実装し、将来のための拡張は入れない。
- **KISS**: まずはシンプルな解決策を選び、複雑化を避ける。
- **DRY**: 重複を避け、共通処理は適切に抽象化する。

## コード品質チェック（推奨）

- 本当に今必要な機能か（YAGNI）
- シンプルで読みやすいか（KISS）
- 重複がないか（DRY）
- 1つの責務に集中しているか

## 実装ルール

- 既存のコード構造と命名規則に従う。
- App Router の基本に従い、Server/Client の責務を混同しない。
- 認証が必要なデータ取得は `user_id` でスコープし、未ログインはリダイレクト。
- AI 呼び出しは `lib/ai/` の薄いラッパー経由（キー未設定時は安全に失敗）。
- Tailwind v4 推奨クラス（`bg-linear-to-*` など）を優先し、警告を避ける。
- **ユーザー入力は必ずバリデーション**: フォーム/URL/リクエストボディ等は必ず検証し、未検証の値をDBや外部APIに渡さない。
- **変更を加えたら必ず型チェック**: 変更後は `npm run type-check` を実行して型の整合性を確認する。
- **分割と構造化**: 可能な限りコンポーネント化し、200行以上のコードは分割を検討する。ディレクトリは責務ごとに分け、`schema` 層や `utils` 層などのまとまりで実装する。
- **UIスタイル（ライトDQ風）**: ダッシュボードの「目標・軸を編集する」リンクのUIを基準に、クエスト風のテイストを維持する。
- **UI基準（DQウィンドウ/ボタン）**: `dq-window` / `dq-title` / `dq-item` / `dq-button` / `dq-button-secondary` / `dq-menu-item` を基準スタイルとして使い、文言は「クエスト/ログ/追加/〜へ」に揃える。
- **UI方針（遷移・アクション項目）**: ボックス付きボタンは原則使わず、サイドバーと同じ「左三角（▶）+ ホバーで三角が右へ移動 + テキスト色変化」の形式で統一する。

## コマンド

```bash
npm run dev        # 開発サーバー
npm run lint       # ESLint
npm run type-check # TypeScript
npm run build      # 本番ビルド
```

---
> Source: [Yamada040/job_hunting_copilot](https://github.com/Yamada040/job_hunting_copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
