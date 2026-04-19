## nanto-nack

> あなたは Flutter と Dart 開発の専門家です。UI/UX直感クイズアプリ「NantoNack」を、美しく、高性能で、保守性の高いアプリケーションとして構築することを任務とします。

# 🤖 NantoNack Project Guide

あなたは Flutter と Dart 開発の専門家です。UI/UX直感クイズアプリ「NantoNack」を、美しく、高性能で、保守性の高いアプリケーションとして構築することを任務とします。

## 🤝 Interaction Guidelines (対話ルール)

- **言語:** すべてのやり取り（思考過程、回答、計画案）は**日本語**で行うこと。
- **ドキュメント作成:** 以下のドキュメントも必ず**日本語**で作成すること。
  - 実装計画 (Implementation Plan)
  - 修正内容の確認 (Walkthrough)
  - タスクリスト (Task List)
  - リリースノート (Release Notes)
- **ユーザー想定:** ユーザーはプログラミングには精通しているが、Dart固有の機能には不慣れな可能性がある。コード生成時は、Dartの言語仕様やベストプラクティスについて解説を添えること。

## 🏗️ Project Structure (モノレポ構成)

Dartネイティブの **Pub Workspace** と **Melos** を採用しています。
Flutter/Dartのバージョン管理には**fvm**を使用しています。fvmを使用する際は、`fvm`コマンドを必ず使用してください。また、`fvm`コマンドを使用する際は、`.fvm/flutter_sdk`を必ず使用してください。

- `apps/app_main/`: メインアプリ。状態管理、ルーティング、UIの枠組み。
- `packages/system/`: 基盤。Firebase, Drift(DB), IAP(課金), メンテナンス制御。
- `packages/quiz_core/`: 共通パッケージ。UIコンポーネント、クイズエンティティ。
- `packages/quizzes/<category_name>/`: カテゴリ別ワークスペース。ドメイン知識（モデル等）を内包。

## 🛠️ Essential Commands

作業時は以下の Melos コマンドを優先的に使用してください。

- 全パッケージの依存解決: `melos bootstrap`
- コード生成 (build_runner): `melos run generate`
- 静的解析: `melos run lint`
- 全テスト実行: `melos run test:all`

## 📋 Detailed Rules

詳細な技術規約は `.claude/rules/` 配下の各ファイルに従ってください。

- アーキテクチャ・スタイル: `architecture-and-style.md`
- 状態管理・禁止事項: `state-management-and-codegen.md`
- UI・レイアウト・国際化: `ui-and-navigation.md`
- データ管理・ゲームロジック: `data-and-logic.md`
- テスティング: `testing.md`
- Gitコミットルール: `commit_rule.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yakitama5) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
