## shou-portfolio

> 当プロジェクトは、Next.js (Reactフレームワーク) を使用して構築されたポートフォリオサイトです。

# プロジェクト構成

当プロジェクトは、Next.js (Reactフレームワーク) を使用して構築されたポートフォリオサイトです。

## 主要な技術スタック

- **フレームワーク**: [Next.js](https://nextjs.org/)
- **UIライブラリ**: [Material-UI (MUI)](https://mui.com/)
- **アニメーション**: [Framer Motion](https://www.framer.com/motion/)
- **言語**: TypeScript

## ディレクトリ構成の概要

- **`/src`**: アプリケーションのソースコードが含まれます。
  - **`/src/app`**: Next.jsのApp Routerに基づいた各ページのコンポーネントが配置されています。
    - **`/api`**: APIルートのエンドポイントが定義されています。
    - **`/career`**: 経歴ページのコンポーネントです。
    - **`/contact`**: お問い合わせフォームのコンポーネントです。
    - **`/outputs`**: 成果物一覧ページのコンポーネントです。
    - **`/projects`**: プロジェクト詳細ページのコンポーネントです。
    - **`/skills`**: スキル一覧ページのコンポーネントです。
  - **`/src/components`**: アプリケーション全体で再利用されるUIコンポーネント（ヘッダー、フッターなど）が格納されています。
- **`/_contents`**: マークダウン形式のコンテンツファイルが格納されています。
  - **`/career`**: 経歴に関するマークダウンファイルです。
  - **`/projects`**: プロジェクトに関するマークダウンファイルです。
- **`/docs`**: プロジェクトの開発関連ドキュメントが格納されています。
  - `requirements.md`: 要件定義書
  - `design.md`: 設計書
  - `tasks.md`: タスク管理表
- **`/public`**: 静的なアセットファイル（画像など）が配置されています。
- **`package.json`**: プロジェクトの依存関係とスクリプトが定義されています。
- **`GEMINI.md`**: Geminiがプロジェクトの構成を把握するためのファイルです。

## 開発ルール

- `README.md`に記載された開発ルールと開発ステップに従ってください。
- `docs`ディレクトリ内のドキュメントを参照し、開発を進めてください。
- 実装とドキュメントの状態は常に一致させてください。
- `docs/tasks.md`のタスクが完了するごとにコミットを行ってください。
- ソースコードを変更した後は、必ず`npm run lint`と`npm run build`を実行し、エラーがないことを確認してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shou-dev19)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shou-dev19)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
