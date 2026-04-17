## learnenglish

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 開発コマンド

### 基本的なコマンド

- `npm run dev` - 開発サーバーを起動（localhost:3000）
- `npm run dev -- --host` - 外部アクセスが必要な場合:
- `npm run build` - 本番用にビルド
- `npm run preview` - 本番ビルドをプレビュー
- `npm run lint` - ESLint でコード品質をチェック

## アーキテクチャ概要

### コンポーネント構造

- **メインコンポーネント**: `src/VocabularyGameApp.jsx` - アプリケーション全体のゲームロジックを含む単一の大きなコンポーネント
- **エントリーポイント**: `src/main.jsx` - React アプリケーションを DOM にマウント
- **重複コンポーネント**: `src/App.jsx` は `VocabularyGameApp.jsx` とほぼ同じ内容（main.jsx では VocabularyGameApp を使用）

### ビルドシステム

- **Vite**: モダンなビルドツールとして使用
- **依存関係の最適化**: lucide-react を含む vendor チャンクと ui チャンクに分割
- **開発サーバー**: ポート 3000、自動ブラウザ起動

### スタイリング

- **Tailwind CSS**: メインのスタイリングフレームワーク
- **カスタム設定**:
  - カスタムアニメーション（pulse）
  - カスタムスケール（scale-102）
  - グラデーション背景
- **PostCSS**: Tailwind 処理用

## 主要機能

### ゲームモード

- **ノーマルモード**: 30 秒の制限時間
- **スピードモード**: 15 秒の制限時間

### 問題形式

- **記述式**: ユーザーが答えを入力（+5pt ボーナス）
- **4 択問題**: 選択肢から選択

### 出題方向

- **日本語 → 英語**: 日本語から英単語を答える
- **英語 → 日本語**: 英単語から日本語の意味を答える
- **例文 → 英単語**: 例文の空欄に入る英単語を答える（銀のフレーズ式）

### データ管理

- **デフォルト語彙**: アプリに 25 個の英単語を内蔵
- **CSV アップロード**:
  - フォーマット: `英単語,日本語,例文,例文の日本語訳`
  - エンコード対応: Shift_JIS（推奨）、UTF-8、EUC-JP
  - 1 行目はヘッダーとして自動スキップ
  - 3 列目・4 列目は例文モード用（省略可）

### スコアリングシステム

- 基本得点: 10pt
- 難易度ボーナス: intermediate +5pt、advanced +10pt
- スピードボーナス: 残り時間に応じて最大 20pt
- コンボボーナス: 5 連続正解以上で倍率アップ（最大 3 倍）
- ヒント未使用ボーナス: +5pt
- 問題形式ボーナス: 記述式 +5pt

## 開発時の注意点

### ファイル構造

- `src/VocabularyGameApp.jsx`: 1160 行の大きなコンポーネント（全機能を含む）
- CSV パースはシンプルな実装（`,`で分割）
- 状態管理は React useState フックのみ使用

### コード品質

- ESLint 設定: React + TypeScript ルール
- 最大警告数: 0（厳格な設定）
- 未使用ディレクティブのレポートを有効

### パフォーマンス

- Vite の最適化設定で lucide-react を明示的にインクルード
- 手動チャンク分割で vendor（react、react-dom）と ui（lucide-react）を分離
- ソースマップは本番では無効

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kagerw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
