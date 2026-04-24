## ar-lp

> このファイルは、このリポジトリのコードを扱う際に Claude Code (claude.ai/code) および他のAIエージェントが参照するための単一の信頼できる情報源（Single Source of Truth）です。

# CLAUDE.md

このファイルは、このリポジトリのコードを扱う際に Claude Code (claude.ai/code) および他のAIエージェントが参照するための単一の信頼できる情報源（Single Source of Truth）です。

## 1. プロジェクト概要

**Tech Connect**
AR（拡張現実）技術を名刺、チラシ、カタログ、商品ラベルなどのマーケティング資料に統合するサービスのランディングページです。アプリ不要のWebAR技術を活用し、ブラウザだけで体験できるARサービスを紹介しています。

## 2. アーキテクチャと技術スタック

### 主要ファイル構造
- `index.html`: シングルページアプリケーション（全てのセクションをインライン記述）
- `style.css`: Tailwind CSS (@layer utilities) + カスタムCSS変数によるスタイル定義
- `organic-particles-simple.js`: Three.jsを使用した背景パーティクルシステム（WebGLフォールバック付き）
- `assets/`: 本番用画像アセット（1MB以下推奨、日本語ファイル名を含む）

### 技術スタック
- **Frontend**: Vanilla HTML/CSS/JavaScript (No Build Process for HTML/JS)
- **CSS**: Tailwind CSS (CDN) + Custom Utilities
- **3D Graphics**: Three.js (r128) - モバイルパフォーマンス最適化済み
- **Animations**: AOS (Animate On Scroll)
- **Forms**: Netlify Forms (スパム保護付き)
- **Fonts**: Google Fonts (Noto Sans JP, Montserrat)

## 3. 開発ワークフロー

### ローカル開発コマンド
- **サーバー起動**: `python3 -m http.server 8000`
  - http://localhost:8000/index.html でアクセス
- **CSS監視・ビルド**: `npx tailwindcss -i style.css -o style.bundle.css --watch`
  - 開発中はCDN版を使用し、本番最適化が必要な場合のみバンドルを使用

### コーディング規約
- **HTML/JS**: 4スペースインデント
- **CSS**: 2スペースインデント
- **命名規則**: 顧客向けの言語に一致する説明的なクラス名（例: `.hero-text-enhanced`）
- **文字コード**: 新規ファイルはASCII推奨だが、既存のUTF-8（日本語テキスト/ファイル名）は保持する

### デザインパターン
- **CSS**: `:root`での変数管理（`--navy`, `--accent-yellow`など）を徹底。
- **レスポンシブ**: モバイルファーストで設計。パーティクル数は画面サイズに応じて調整（2000/3500/5000個）。

## 4. テストと品質保証

- **ブラウザテスト**: Chrome, Safari, モバイルエミュレーターで確認。
- **WebGLフォールバック**: `.hero-static-bg` クラスが正しく機能するか確認（ハードウェアアクセラレーション無効化テスト）。
- **Lighthouse**: 視覚的な変更を行う際はLighthouseスコアを確認し、大幅な低下（5ポイント以上）がないか注意する。
- **AR機能**: QRコード/リンクからAR機能が正しく起動するか実機確認を行う。

## 5. ビジネスコンテキスト（サービス仕様）

### 料金プラン
1.  **STANDARD PACKAGE**: ¥25,000 (キャンペーン価格: ¥15,000)
    -   マーカー登録、素材配置、テスト、6ヶ月間のサーバー保守込み
2.  **保守プラン** (7ヶ月目以降):
    -   **Light**: ¥800/月 (ホスティングのみ)
    -   **Standard**: ¥5,000/月 (ホスティング + 小規模更新 + サポート)
    -   **Premium**: ¥10,000/月 (ホスティング + 更新x2 + 優先サポート + レポート)
3.  **スポットメニュー**: 契約者以外向けの単発修正/レポート

※ すべての価格は**税込表示**で統一してください。

## 6. デプロイとGit運用

### デプロイ
- **GitHub Pages**: `techconnect-em.github.io/ar_lp` (メイン)
- **Netlify**: フォーム機能のバックエンドとして利用

### Gitワークフロー
- すべての変更は **Commit & Push** (`git push origin main`) を行う。
- コミットメッセージは命令形・英語・センテンスケース（例: `Improve hero particle density`）。
- `.DS_Store` はコミットしない。

## 7. 更新ガイドライン

- **スタイル変更**: パーティクルシステムへの影響を考慮し、必ずモバイル実機またはエミュレーターで確認する。
- **ナビゲーション**: デスクトップ、モバイル、フッターの3箇所を同期して更新する。
- **ヒーローコピー**: メインタイトルとサブ要素の一貫性を保つ。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/techconnect-em) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
