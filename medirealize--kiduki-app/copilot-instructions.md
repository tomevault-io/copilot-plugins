## kiduki-app

> - **KiDuKi**: 診察前の患者の「沈黙（未言語化の思考）」をAI対話で言語化し、診察の質を上げるWebアプリ

# KiDuKi プロジェクト共通ルール

## プロジェクトの位置づけ
- **KiDuKi**: 診察前の患者の「沈黙（未言語化の思考）」をAI対話で言語化し、診察の質を上げるWebアプリ
- 診断・医療判断は一切行わない。「気づき」と「整理」に特化する。
- ターゲット: 40〜70歳の外来患者。シンプルで温かみのあるUXが必須。

## 技術スタック
- **Frontend**: Next.js (App Router), Tailwind CSS, shadcn/ui
- **AI**: OpenAI API (GPT-4o / gpt-4o-mini)
- **State**: React Hook Form（フォーム入力）
- **Deploy**: Vercel (Serverless Functions)

## コーディング規約
- TypeScript を基本とする。型は明示し、`any` は避ける。
- コンポーネント・API・定数は `docs/prd.md` および実装設計書の用語（Q1, Q2, Q3, SUM 等）に合わせる。
- スタイルは Tailwind を優先。デザイントークン（色・フォントサイズ）は一箇所で管理する。

## UX・アクセシビリティ
- フォントサイズは読みやすく（本文 16px 以上推奨）、タップ領域は 44px 以上を目安にする。
- 医療系のため、文言は「診断しない」「補助ツールである」旨を適宜明示する。
- エラーメッセージ・空状態は温かみのあるトーンで統一する。

## セキュリティ・倫理
- 患者入力は診断に用いない。プロンプト設計で「気づき・整理」に限定する。
- API Key 等の秘密情報は環境変数に格納し、リポジトリにコミットしない。
- 個人を特定できる情報の扱いは PRD・要件定義に従う。

## 参照ドキュメント
- 要件・機能: `docs/prd.md`
- 実装詳細: プロジェクト内の実装設計書・問い生成アルゴリズム設計を参照する。

---
> Source: [Medirealize/kiduki-app](https://github.com/Medirealize/kiduki-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
