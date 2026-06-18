## tune

> TUNE は「使うほど育つ AI アシスタント」。業務を依頼→成果物にフィードバック→知識が「引き出し」として蓄積され、次回以降の品質が向上する Web アプリケーション。

# TUNE — エージェント向けガイド

TUNE は「使うほど育つ AI アシスタント」。業務を依頼→成果物にフィードバック→知識が「引き出し」として蓄積され、次回以降の品質が向上する Web アプリケーション。

> このファイルは **地図** です。詳細は各参照先を読んでください。

---

## ディレクトリマップ

| パス | 役割 |
|---|---|
| `CLAUDE.md` | ドメインモデル・設計方針・DB スキーマ・コーディング規約の **単一情報源** |
| `docs/ARCHITECTURE.md` | レイヤー構造・パッケージ依存のトップレベルマップ |
| `docs/design-overview.md` | 設計書 v4（プロダクト概要・理論基盤・改定履歴） |
| `docs/requirements.md` | 要件定義書 |
| `docs/development-tasks.md` | 開発タスク（Week 別） |
| `docs/ux-copy-and-labels.md` | 非エンジニア向け UX コピー・種別ラベル仕様 |
| `docs/design-specification.md` | デザイン仕様（画面構成・トークン・コピー統合・レビュー用） |
| `docs/drawer-extractor-prompt.md` | 引き出し抽出プロンプト仕様 |
| `docs/brand-guidelines.md` | ブランドガイドライン |
| `migrations/` | SQLite マイグレーション SQL |
| `scripts/` | 開発補助スクリプト（アーキテクチャ検証など） |
| `src/domain/` | ドメイン層（pure TypeScript、外部依存禁止） |
| `src/application/` | アプリケーション層（ユースケース） |
| `src/infrastructure/` | インフラ層（SQLite, Claude API 等） |
| `src/registry/` | DI コンテナ（手動注入） |
| `src/app/` | Next.js App Router（UI + API Routes） |
| `src/components/` | React コンポーネント |

## レイヤー依存ルール

```
UI (src/app/, src/components/)
  ↓
Application (src/application/)
  ↓
Domain (src/domain/)  ←  Infrastructure (src/infrastructure/)
```

- **domain/ は外部ライブラリを import しない**（pure TypeScript のみ）
- application/ は domain/ のインターフェースを inject で受け取る
- infrastructure/ は domain/ のインターフェースを実装する
- UI は application/ のユースケースを呼ぶだけ。ビジネスロジックを書かない

## 黄金ルール（機械的に検証される）

1. **domain/ に `node_modules` からの import 禁止** — `scripts/check-architecture.ts` が検出
2. **レイヤー逆流禁止** — domain → application/infrastructure の import は違反
3. **テストなしの新規ロジック禁止** — TDD（レッド→グリーン→リファクタ）を守る
4. **設計判断はリポジトリにコミット** — Slack や口頭で決めた内容も docs/ に残す
5. **UI 文言は内部用語を出さない** — `docs/ux-copy-and-labels.md` を参照

## 開発ワークフロー

```
1. タスク選定    → docs/development-tasks.md から選ぶ
2. ブランチ作成  → feature/短い識別子 or fix/短い識別子
3. TDD で実装    → レッド→グリーン→リファクタ
4. 自己レビュー  → npm run check:arch && npm run test
5. PR 作成      → `gh pr create --base main`（pr-creation スキル参照。マージ先は既定で **main**）
6. マージ       → レビュー通過後
```

## エージェント定義（.cursor/agents/）

| エージェント | 役割 | いつ使う |
|---|---|---|
| `architect` | 設計レビュー・依存確認 | 「アーキテクトとして見て」 |
| `implementer` | TDD で実装 | 実装依頼時 |
| `pm` | 要件整理・タスク分解 | 機能追加・スプリント計画 |
| `tester` | テスト設計・コード作成 | テスト依頼・レビュー時 |

## ルール（.cursor/rules/）

| ルール | 適用条件 | 内容 |
|---|---|---|
| `frontend-screen-composition` | 常時適用 | 画面構成・文言・視覚ルール |
| `architecture` | `src/domain,application,infrastructure/**` | DDD・依存方向 |
| `tdd-implementation` | `src/**/*.ts`, `**/*.test.ts` | TDD サイクル |
| `git-commit-branch` | `src,docs,migrations/**` | コミット形式・ブランチ命名 |
| `self-review` | 常時適用 | 変更完了前の自己チェック |
| `knowledge-management` | `docs/**`, `CLAUDE.md`, `AGENTS.md` | ドキュメント鮮度管理 |

## 主要スキル（.cursor/skills/）

| スキル | いつ使う |
|---|---|
| `clean-architecture` | 設計レビュー・リファクタ |
| `tdd-t-wada` | 実装時（TDD 手順の詳細） |
| `testing-t-wada` | テスト設計・カバレッジ検討 |
| `faceted-prompting` | プロンプト構造化 |
| `git-workflow` | ブランチ・コミット操作 |
| `pr-creation` | PR 作成手順 |
| `pm-tasks` | 要件整理・タスク分解 |
| `agent-self-review` | PR 前の自己レビューループ |
| `cleanup-patrol` | 技術的負債の検出・修正 |

## npm スクリプト

```bash
npm run dev          # 開発サーバー起動
npm run build        # ビルド
npm run test         # テスト実行（vitest）
npm run test:watch   # テスト監視モード
npm run check:arch   # アーキテクチャ検証
npm run lint         # ESLint
```

---
> Source: [mism-mism/tune](https://github.com/mism-mism/tune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
