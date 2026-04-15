## csv-test-takigawa

> これは、インターンシップの課題として設計されたユーザー管理アプリケーションです。Next.jsのフロントエンドとLaravelのバックエンドで構成されています。このアプリケーションには、セキュリティの脆弱性、パフォーマンスのボトルネック、不適切なコーディングプラクティスなど、意図的に様々な問題が含まれており、それらを特定して修正することが目的です。

# GEMINI.md

## プロジェクト概要

これは、インターンシップの課題として設計されたユーザー管理アプリケーションです。Next.jsのフロントエンドとLaravelのバックエンドで構成されています。このアプリケーションには、セキュリティの脆弱性、パフォーマンスのボトルネック、不適切なコーディングプラクティスなど、意図的に様々な問題が含まれており、それらを特定して修正することが目的です。

**技術スタック:**

*   **フロントエンド:** Next.js, TypeScript, Tailwind CSS, shadcn/ui
*   **バックエンド:** Laravel 10, PHP 8.1, MySQL 8.0
*   **インフラ:** Docker, Docker Compose

## ビルドと実行

1.  **アプリケーションの起動:**
    ```bash
    docker-compose up -d
    ```

2.  **データベースの初期化 (バックエンドコンテナ内で実行):**
    *   テスト環境 (ユーザー1,000人):
        ```bash
        docker compose exec backend bash
        php artisan app:init-database-test
        ```
    *   本番環境 (ユーザー1,000,000人):
        ```bash
        docker compose exec backend bash
        php artisan app:init-database-production
        ```

3.  **アプリケーションへのアクセス:**
    *   フロントエンド: http://localhost:3000
    *   バックエンドAPI: http://localhost:8000/api

**主要なコマンド:**

*   **バックエンド (`backend` ディレクトリ内):**
    *   テスト実行: `composer test`
    *   リント: `composer pint`
*   **フロントエンド (`frontend` ディレクトリ内):**
    *   開発サーバー起動: `npm run dev`
    *   本番ビルド: `npm run build`
    *   リント: `npm run lint`
    *   型チェック: `npm run typecheck`

## 開発規約

*   **バックエンド:**
    *   Laravelフレームワークの規約に従います。
    *   APIルートは `backend/routes/api.php` で定義されています。
    *   データベース操作にはEloquent ORMを使用します。
    *   依存関係はComposerで管理します。
*   **フロントエンド:**
    *   Next.jsフレームワークの規約に従います。
    *   依存関係はnpmで管理します。
*   **課題とリスク:**
    *   既知の問題リストは `docs/backlog/issues.md` にあります。
    *   セキュリティと保守性のリスクは `docs/risk/security_and_maintainability_risks.md` に文書化されています。
    *   UI/UXに関する問題は `docs/ui_ux_issues.md` に文書化されています。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bsj-m-takigawa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bsj-m-takigawa)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
