## takumiokayasu-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

このリポジトリは `takumiokayasu.github.io` のGitHub PagesプロジェクトでReact + TypeScript + Viteを使用したSPAベースのポートフォリオサイトです。現在は開発段階にあり、デモ用のHTMLファイルとスキルデータのJSONファイルが含まれています。

## 開発環境

### Dev Container
このプロジェクトは`.devcontainer`を使用した開発環境が構築されています:
- **ランタイム**: Bun (高速JavaScriptランタイム)
- **フレームワーク**: React 19.1.0 + TypeScript
- **ビルドツール**: Vite 7.0.4
- **リントツール**: ESLint 9.30.1 + TypeScript ESLint
- **コードフォーマッター**: Biome (VS Code拡張機能)
- **プラグイン**: @vitejs/plugin-react-swc (高速リフレッシュ)

### 開発環境の起動
```bash
# VS CodeでDev Containerを起動
# 「Reopen in Container」を実行
```

## プロジェクト構造

```
/
├── .devcontainer/     # Dev Container設定
├── src/               # React アプリケーションのソースコード
│   ├── App.tsx        # メインアプリケーション
│   ├── main.tsx       # エントリーポイント
│   ├── assets/        # アセット (react.svg)
│   └── *.css          # スタイルシート
├── public/            # 静的アセット (vite.svg)
├── mock/              # デモデータ
│   ├── mock.html      # HTMLデモページ
│   └── skills.json    # スキルデータ
├── dist/              # ビルド出力
└── index.html         # HTMLテンプレート
```

## 開発コマンド

### 現在利用可能なコマンド
```bash
# 開発サーバー起動
npm run dev

# プロダクションビルド
npm run build

# ビルドプレビュー
npm run preview

# リント実行
npm run lint

# 依存関係のインストール
npm install
```

### テスト実行
現在テストフレームワークは設定されていません。Vitestの導入を推奨します。

## アーキテクチャ

### 現在の技術スタック
- **フロントエンド**: React 19.1.0 + TypeScript
- **ビルドツール**: Vite 7.0.4
- **スタイリング**: CSS (Tailwind CSSはmock/mock.htmlでのみ使用。srcへの適用は [要確認])
- **高速リフレッシュ**: SWC (Speedy Web Compiler)

### TypeScript設定
- プロジェクトレベル: `tsconfig.json` (参照設定)
- アプリケーション: `tsconfig.app.json`
- Node.js: `tsconfig.node.json`

### ESLint設定
- TypeScript ESLint + React Hooks + React Refresh
- 設定ファイル: `eslint.config.js`
- 推奨設定を使用、必要に応じてtype-aware lintingに対応可能

## データ構造

### スキルデータ (mock/skills.json)
プロフィールページで使用するスキルデータが構造化されています:
- **categories**: スキルカテゴリ配列
  - programming-languages
  - frameworks-libraries
  - databases
  - cloud-devops
  - tools-platforms
- **skills**: 各スキルの詳細情報 (name, level, years, description)

### デモページ (mock/mock.html)
Tailwind CSSを使用したデモページが含まれています。本格的なReactアプリケーションへの移行時の参考になります。

## GitHub Pages設定

- ビルド出力は`dist/`ディレクトリに配置
- GitHub Actionsでの自動デプロイ設定を推奨
- SPAのため、GitHub Pagesでのルーティング設定が必要

## 品質保証

### コード品質チェックリスト (報告前に必ず全項目確認)

| チェック項目 | コマンド | 合格条件 |
|------------|---------|---------|
| ESLint | `npm run lint` | エラー0件 |
| TypeScript型チェック | `npm run build` | エラー0件 |
| Biome | VS Code保存時に自動実行 | 警告0件 |

**報告前に全チェックをパスすること。未確認での完了報告は禁止。**

### テスト戦略
- ユニットテスト: Vitest推奨 (未設定)
- E2Eテスト: Playwright推奨 (未設定)
- 現在テストフレームワークは設定されていません

## 🚨 禁止事項

| 操作 | 理由 |
|------|------|
| `dist/` ディレクトリの手動編集 | ビルド出力は自動生成。手動変更は `npm run build` で上書きされる |
| `mock/` ディレクトリのデータをsrcに直接コピー | 移行時は必ずReactコンポーネントとして再実装する |
| `package.json` のバージョン直接書き換え | 必ず `npm install <pkg>` または `npm update` を使用 |
| テストなしでのコンポーネント追加 | Vitestが未設定でも、手動動作確認を必ず実施して報告すること |
| 存在しないパッケージのimport | 必ず `package.json` の `dependencies` / `devDependencies` に存在するか確認する |
| `tsconfig.json` の直接編集 | `tsconfig.app.json` / `tsconfig.node.json` を目的に応じて編集する |

## スキル参照

複雑な作業を行う際は以下のグローバルスキルを参照すること:

| スキル | 用途 | 参照タイミング |
|--------|------|--------------|
| `test-driven-development` | コンポーネント・ロジック実装 | 新規実装前 |
| `systematic-debugging` | バグ調査・型エラー解消 | 原因不明のエラー発生時 |
| `interface-first-design` | コンポーネントAPI設計 | 新規コンポーネント設計時 |

---
> Source: [TakumiOkayasu/takumiokayasu.github.io](https://github.com/TakumiOkayasu/takumiokayasu.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
