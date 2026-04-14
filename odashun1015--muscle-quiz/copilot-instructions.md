## muscle-quiz

> このドキュメントは、筋肉クイズアプリケーションの技術スタック、アーキテクチャ、および主要コンポーネントの概要を説明します。

# 筋肉クイズアプリケーション分析

このドキュメントは、筋肉クイズアプリケーションの技術スタック、アーキテクチャ、および主要コンポーネントの概要を説明します。

## 技術スタック

*   **フレームワーク**: [Vue.js](https://vuejs.org/) 3 (バージョン ^3.5.13)
*   **ビルドツール**: [Vite](https://vitejs.dev/) (バージョン ^6.2.4)
*   **言語**: [TypeScript](https://www.typescriptlang.org/) (バージョン ~5.8.0)
*   **パッケージマネージャー**: npm (`package.json` と `package-lock.json` から推測)
*   **開発サーバー**: `vite`
*   **リンティング/型チェック**: `vue-tsc`

## プロジェクト構造とアーキテクチャ

このアプリケーションは、標準的な Vue 3 + Vite のプロジェクト構造に従っています。

*   **`src/`**: メインのソースコードディレクトリ。
    *   **`main.ts`**: アプリケーションのエントリーポイント。Vue アプリを初期化し、ルートコンポーネント (`App.vue`) をマウントします。
    *   **`App.vue`**: ルートコンポーネント。アプリケーション全体のの状態を管理し、現在のゲームの状態に基づいてさまざまな画面（タイトル、ゲーム、結果）を描画します。
    *   **`components/`**: アプリケーションで使用されるすべての Vue コンポーネントが含まれています。
        *   `TitleScreen.vue`: クイズの初期画面。
        *   `GameScreen.vue`: ユーザーが質問に答えるメイン画面。
        *   `ResultScreen.vue`: クイズの結果を表示する画面。
        *   `MuscleCanvas.vue`: `FrontMuscleCanvas` と `BackMuscleCanvas` を組み合わせ、体の全体図を表示します。
        *   `FrontMuscleCanvas.vue`: 体の前面の筋肉をSVGで描画します。
        *   `BackMuscleCanvas.vue`: 体の背面の筋肉をSVGで描画します。
        *   `QuizOptions.vue`: クイズの選択肢を表示し、ユーザーの回答を処理します。
        *   `HelloWorld.vue`, `TheWelcome.vue`, `WelcomeItem.vue`: Vueプロジェクト初期生成時のサンプルコンポーネントであり、アプリケーションのコア機能には直接関係ありません。
    *   **`composables/`**: コアアプリケーションロジックが含まれています。
        *   `useQuizGame.ts`: 状態管理、問題生成、ゲームフロー制御など、ゲームロジック全体をカプセル化する Vue コンポーザブル。これは、アプリケーションの機能の中心です。
    *   **`data/`**: クイズの生データが含まれています。
        *   `muscles.json`: 筋肉データ（ID、名前、カテゴリなど）を含む JSON ファイル。
    *   **`types/`**: TypeScript の型定義が含まれています。
        *   `index.ts`: アプリケーション全体で使用されるデータ構造（`Muscle`、`QuizSettings`、`GameState` など）を定義します。
    *   **`assets/`**: CSS や画像などの静的アセットが含まれています。
*   **`public/`**: ビルド出力に直接コピーされる静的アセットが含まれています。
*   **`dist/`**: ビルド出力ディレクトリ。
*   **設定ファイル**:
    *   `vite.config.ts`: プラグイン（`@vitejs/plugin-vue`）、パスエイリアス（`@`）、ビルド設定など、Vite の設定。
    *   `tsconfig.json`: TypeScript の設定。
    *   `package.json`: プロジェクトのメタデータ、依存関係、およびスクリプト。

## アプリケーションフローと状態管理

このアプリケーションは、シンプルな状態駆動アーキテクチャを持つシングルページアプリケーション（SPA）です。

1.  **初期化**: `main.ts` は、`App.vue` をルートコンポーネントとして Vue アプリインスタンスを作成します。
2.  **状態管理**: コアロジックと状態は、`useQuizGame.ts` コンポーザブル内で管理されます。このコンポーザブルは、Vue のリアクティビティシステム（`ref`、`computed`、`reactive`）を使用してゲームの状態を管理します。
3.  **ゲームの状態**: `useQuizGame.ts` の `gameState` オブジェクトは、現在の画面（`title`、`game`、`result`）、クイズ設定、問題、ユーザーの進行状況、および結果を追跡します。
4.  **コンポーネントの相互作用**:
    *   `App.vue` は `useQuizGame` コンポーザブルを呼び出して、ゲームの状態と関数を取得します。
    *   次に、`v-if` ディレクティブを使用して、適切な画面コンポーネント（`TitleScreen`、`GameScreen`、または `ResultScreen`）を条件付きでレンダリングします。
    *   画面コンポーネントはイベント（`@start-game`、`@select-answer` など）を発行し、`App.vue` によって処理されます。これにより、`useQuizGame` コンポーザブルから対応する関数が呼び出され、ゲームの状態が更新されます。

## 主な機能と実装

*   **クイズ生成**: `useQuizGame.ts` の `generateQuestions` 関数は、選択されたカテゴリと設定に基づいて `muscles.json` から筋肉をランダムに選択することにより、クイズの問題を作成します。
*   **問題の選択肢**: `generateChoices` 関数は、各問題の多肢選択式の選択肢を作成し、1 つの正解といくつかのランダムな不正解を保証します。
*   **リアクティビティ**: `gameState` オブジェクトの変更に応じて UI が自動的に更新されます。これは Vue のリアクティビティシステムのおかげです。たとえば、`gameState.currentScreen` が変更されると、`App.vue` のビューが新しい画面に切り替わります。
*   **コンポーネントベースのアーキテクチャ**: UI は再利用可能なコンポーネントに分割されているため、コードはモジュール化されており、保守が容易です。

## 利用可能なコマンド

`package.json` に定義されている主要なスクリプトは以下の通りです。

*   `npm run dev`: 開発モードでアプリケーションを起動します。ホットリロードが有効になります。
*   `npm run build`: 本番用にアプリケーションをビルドします。`dist` ディレクトリに最適化されたファイルが生成されます。
*   `npm run preview`: ビルドされたアプリケーションをローカルでプレビューします。
*   `npm run type-check`: TypeScriptの型チェックを実行します。

## ビルドとデプロイ

*   `npm run build` スクリプトは `vite build` をトリガーし、アプリケーションを本番用に `dist/` ディレクトリにバンドルします。
*   `.github/workflows/deploy.yml` ファイルは、`vite.config.ts` の `base` パス設定を考えると、プロジェクトが GitHub Actions を使用して、おそらく GitHub Pages に自動デプロイされるように設定されていることを示唆しています。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/odashun1015)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/odashun1015)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
