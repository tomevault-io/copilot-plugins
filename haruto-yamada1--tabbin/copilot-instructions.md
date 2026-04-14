## tabbin

> This file provides guidance to Gemini (Antigravity) when working with code in this repository.

# GEMINI.md

This file provides guidance to Gemini (Antigravity) when working with code in this repository.

このファイルは、このリポジトリでコードを作業する際に Gemini (Antigravityエージェント) にガイダンスを提供します。

## 基本原則

- すべてのアウトプットは日本語で、初心者にも理解しやすい丁寧な表現を使用する。
- 不明点があれば意図を確認し、わからないまま進めない。
- コードを書く前に既存実装をレビューし、その動作と目的を言語化する。
- タスクは小さなステップに分解し、Agentic Mode機能（`task_boundary`等）を活用して進める。
- 関数型・宣言型パターンを好み、クラスの使用を避ける。
- セキュリティとパフォーマンスを常に意識し、潜在的リスクがあれば早期に指摘する。
- 行動を励ますポジティブなトーンを保ち、学習意欲を高める。

## プロジェクト概要

TABBIN（タビン）は、React、TypeScript、WXTフレームワークを使用したインテリジェントなタブ管理のためのChrome拡張機能です。スマートな分類とプロジェクトベースの整理を通じて、ユーザーがブラウザタブを整理するのを支援します。メインのエントリーポイントは `entrypoints/` にあり、ドメイン機能は `features/` に、共有コンポーネントは `components/` に配置されています。

## 開発コマンド (Bunを利用)

### コア開発
- `bun install` - 依存関係のインストール（CIはNode 22とBun 1.2.8を使用）
- `bun run dev` / `bun run dev:firefox` - Chrome/Firefox向けホットリロード付き開発サーバーを起動
- `bun run build` / `bun run build:firefox` - プロダクション拡張機能をビルド
- `bun run zip` / `bun run zip:firefox` - 配布可能な拡張機能zipを作成

### コード品質とテスト
- `bun run compile` - TypeScript型チェック（`tsgo --noEmit`）
- `bun run test` / `bun run test:coverage` - Vitestでユニットテストを実行（必要に応じてカバレッジ取得）
- `bun run e2e` - PlaywrightでE2Eテストを実行
- `bun run quality` - フォーマット、Lint(Biomeチェック)、コンパイル検証、各種テストを一括実行

## Agent Notify (Completion Gate)

AIエージェントがタスクを完了としてユーザーに報告する前の**必須ステップ**：

1. `bun run quality` を実行する。
2. もし失敗した場合はエラーを修正し、パスするまで再実行する。
3. `bun run test:coverage` を実行する。
4. カバレッジが `100` でない場合、テストを追加・修正し `100` になるまで再実行する。
5. 両方のコマンドがパスし、カバレッジが `100` になるまで完了報告（`notify_user`等）を行わない。

※環境やツールの問題（リポジトリのコードと無関係なランタイムパニックなど）でブロックされた場合は、完了報告ではなくブロッカーとして明示的に報告すること。

## アーキテクチャ概要

### 拡張機能構造
- **WXTフレームワーク**: manifest v3を使用したモダンなWeb拡張機能開発。
- **エントリーポイント**: `background.ts`, `options/`, `saved-tabs/`, `changelog/`
- **機能ベースアーキテクチャ**: ドメイン機能（例: `features/options`, `features/saved-tabs`）で整理。

### 主要システム
- **ストレージアーキテクチャ** (`lib/storage/`): TabGroup管理、カテゴリシステム、プロジェクト管理、設定、マイグレーション。
- **表示モード** (`lib/view-modes/`): ドメインモードとカスタムプロジェクトモード。
- **コア機能**: キーワードベース自動分類、ドラッグ&ドロップ整理(@dnd-kit)、あいまい検索(Fuse.js)、コンテキストメニュー統合。

### コンポーネントシステム
- **shadcn/ui + Radix UI**: コンポーネント基盤。
- **Tailwind CSS 4.x**: カスタムデザインシステムを持つユーティリティファーストスタイリング。
- **テーマサポート**: システム設定検出によるダーク/ライトモード。

## コードスタイルと標準

### フォーマットとLint（Biome）
- 2スペースインデント、80文字行幅。
- JS/TSにシングルクォート、セミコロンは必要な場合のみ（`asNeeded`）。
- import整理有効（Biomeにお任せ）。

### TypeScriptとReact
- ストリクトモード有効 (`strict: true`)、`any` 型の使用を禁止し `unknown` と型ガードを利用。
- `var` は禁止。再代入が不要な値は `const` を使用。
- 暗黙の型推論に依存せず、型注釈を明示する。
- Enums の代わりにリテラルユニオン型を使用。
- `class` 宣言およびクラスベースのコンポーネントを禁止し、関数型・宣言型パターンを徹底。
- ファイル命名規則: Reactコンポーネントは `PascalCase.tsx`、ユーティリティ等は `camelCase.ts`。
- ROROパターン（Receive an Object, Return an Object）を適切に採用。
- フラグや状態の変数名は `isLoading`、`hasError` など補助動詞を用いた説明的な命名とする。
- コンポーネントや関数にはJSDocコメントを付与し、目的・引数・戻り値を明示する。

## タスク管理・オーケストレーション (Agentic Mode)

タスクを実行する際は、機能ごとのモード切替（`task_boundary`）とアーティファクト作成を活用して進めます。

### 1. プランニング (PLANNING)
- 非自明なタスクは必ず `PLANNING` モードから開始する。
- 最初に既存コードやドキュメントを読み込み、必要であれば `implementation_plan.md` アーティファクトを作成して方針をユーザーに確認する。

### 2. 実行と進捗管理 (EXECUTION)
- 自動生成・管理される `task.md` のチェックリストを更新しながら実装を進める。
- 未完了 `[ ]` -> 進行中 `[/]` -> 完了 `[x]` のステータスを反映させながら段階的に作業する。
- メインの文脈を綺麗に保つため、機能の探索などは適宜サブタスクとして分散する。

### 3. 検証 (VERIFICATION)
- 実装後は `VERIFICATION` モードに移行し動作確認を行う。E2Eテストや `bun run quality` を通して正しさを証明する。
- 検証結果や変更内容は分かりやすい形で `walkthrough.md` 等にまとめ、画面キャプチャ等を活用してユーザーに成果を提示する。

### 4. 自律的バグ修正
- バグが報告された場合、ログや失敗したテストを確認し、即座に修正案を立案して自律的に直す。

## Gitとコミットルール
- 最近の履歴に合わせて簡潔な件名（多くは日本語）を使用する。
- PRは `main` ターゲットとし、変更内容の要約とローカル環境でのバリデーション確認を記載する。
- UI変更がある場合はスクリーンショットやGIFを添付する。
- `lefthook` により、コミット時に `biome check --write`、プッシュ時に `bun run quality` が走るため、常にローカルでエラーなく通る状態を保つ。

## コア原則
- **シンプル第一**: 変更は可能な限り単純に。影響範囲は最小限に。
- **怠らない**: 根本原因を解決する。応急処置は禁止。シニア水準を保つ。
- **最小影響**: 必要な箇所だけ変更し、新たなバグを生まない。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haruto-yamada1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/haruto-yamada1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
