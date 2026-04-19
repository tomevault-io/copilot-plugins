## profit-calendar

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 前提
- 勝手に開発サーバーを起動しないでください
- 勝手にdevサーバーを起動しないで
- 勝手にpushしないで

## プロジェクト概要
- 利益カレンダー (profit-calendar) - React + TypeScript + Viteで構築されたWebアプリケーション
- SWCを使用した高速開発環境

## 開発コマンド

### 開発サーバー起動
```bash
npm run dev
```

### ビルド
```bash
npm run build
```

### Linting
```bash
npm run lint
```

### プレビュー（ビルド後の確認）
```bash
npm run preview
```

### 型チェック
ビルドコマンドに含まれています（`tsc -b`）

## アーキテクチャ概要
- **フレームワーク**: React 19
- **ビルドツール**: Vite + @vitejs/plugin-react-swc
- **型システム**: TypeScript 5.7
- **エントリーポイント**: src/main.tsx → src/App.tsx
- **スタイリング**: CSS + CSS変数によるデザインシステム
- **チャート**: Chart.js + react-chartjs-2
- **エンコーディング**: encoding-japanese（Shift-JIS対応）

## プロジェクト構造
- `/src` - ソースコード
  - `main.tsx` - アプリケーションのエントリーポイント
  - `App.tsx` - メインコンポーネント
  - `/components` - 再利用可能なUIコンポーネント
    - `/ui` - 汎用UIコンポーネント（Modal, Toast, ConfirmDialog）
    - `/MonthlyReport` - 月次レポート関連コンポーネント
  - `/contexts` - React Context（UIContext, ThemeContext）
  - `/hooks` - カスタムReact Hooks（useCallback/useMemoによるパフォーマンス最適化採用）
  - `/utils` - 汎用的なユーティリティ関数
  - `/types` - TypeScriptの型定義
  - `/styles` - CSSファイル（デザインシステム準拠）
  - `/services` - データベース・バックアップサービス
  - `/constants` - アプリケーション定数
  - `/assets` - 静的リソース
- `/public` - 公開ディレクトリ

## アプリケーション仕様
- **目的**: 取引収益記録カレンダーアプリ
- **主要機能**:
  - SBI証券などのCSVファイルをD&Dでインポート（プレビュー・承認機能付き）
  - 月間カレンダー表示（日別損益・取引件数表示）
  - 日付選択による詳細取引履歴表示
  - IndexedDBによるローカルデータ保存
  - 年間収益チャート表示
  - 月次レポート機能（基本指標・トレンド・リスク・銘柄分析）
  - データバックアップ・復元機能
  - テーマ設定（ライト・ダーク・カスタム）
  - 取引履歴の一括削除機能

## 技術仕様
- **データベース**: IndexedDB
- **CSVフォーマット**: Shift-JIS エンコーディング対応
- **対応取引種別**: 現物売買・信用取引（返済売・返済買）
- **主要パッケージ**:
  - uuid, @types/uuid（ID生成）
  - chart.js, react-chartjs-2（チャート表示）
  - encoding-japanese（文字エンコーディング）

## 注意事項
- Reactベストプラクティスに従ったディレクトリ構造を採用
- テストフレームワークは未設定
- CSVインポート機能はShift-JISエンコーディングを前提
- CSS設計システムに従った統一的なスタイル管理
- `!important`の使用禁止（CSS_DESIGN_SYSTEM.mdを参照）
- 型安全性重視: TypeScript型アサーション（`as`）の使用を極力避け、型ガード関数による実行時型チェックを採用

## CSS設計システム
詳細は`CSS_DESIGN_SYSTEM.md`を参照。主要な原則：
- CSS変数によるデザイントークン管理
- `!important`使用禁止
- BEM記法の採用推奨
- レスポンシブデザイン（768px以下をモバイル基準）
- 統一されたz-index管理

## UI/UX システム
- モダンUIシステム導入済み（ConfirmDialog, Toast, Modal）
- テーマ切り替え機能（ライト・ダーク・カスタムテーマ）
- レスポンシブデザイン対応
- 取引フォーム: 必須項目を上部配置、詳細項目をトグル表示
- フォームボタン下部固定でスクロール改善
- null値の適切な表示処理（「未入力」「不明」表示）

## デプロイ設定
- GitHub Pages対応（`npm run deploy`）
- リポジトリ: https://github.com/ivgtr/profit-calendar.git
- 公開URL: https://ivgtr.github.io/profit-calendar/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivgtr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
