## realestate-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**realestate-app** — Supabase 認証付きの不動産検索 Web アプリ。メール＋パスワードで会員登録・ログインし、物件一覧をカード形式で閲覧できる。

- フロントエンド: **React 18 + Vite**
- 認証 / DB: **Supabase** (Auth)
- ルーティング: **React Router v6**
- スタイル: **CSS Modules** (各ページに `*.module.css`)

## コマンド

```bash
npm run dev      # 開発サーバー起動 (http://localhost:5173)
npm run build    # プロダクションビルド
npm run preview  # ビルド成果物のプレビュー
```

## アーキテクチャ

### 認証フロー

```
App.jsx
 └─ AuthProvider (AuthContext)   ← Supabase セッションをグローバル管理
     └─ BrowserRouter
         ├─ /login       → LoginPage
         ├─ /register    → RegisterPage
         └─ /properties  → PrivateRoute → PropertiesPage
```

`AuthContext` が `supabase.auth.onAuthStateChange` を購読し、`user` と `loading` をアプリ全体に提供する。`PrivateRoute` は `loading === true` の間は何も描画せず、セッション確認完了後に未ログインなら `/login` へリダイレクトする。

### ディレクトリ構成

```
src/
├── lib/supabaseClient.js     # createClient の単一インスタンス
├── contexts/AuthContext.jsx  # user, loading, signIn, signUp, signOut
├── components/PrivateRoute.jsx
└── pages/
    ├── LoginPage.jsx / AuthPage.module.css
    ├── RegisterPage.jsx      # AuthPage.module.css を共有
    └── PropertiesPage.jsx / PropertiesPage.module.css
```

### 環境変数

`.env` に記載（Git 管理外）。Vite では `VITE_` プレフィックスが必須。

```
VITE_SUPABASE_URL=...
VITE_SUPABASE_ANON_KEY=...
```

### 物件データ

現在は `PropertiesPage.jsx` 内のダミーデータ (`DUMMY_PROPERTIES` 配列)。将来 Supabase の `properties` テーブルに移行する際はここを `supabase.from('properties').select()` に置き換える。

## デプロイ情報

- 本番URL：https://realestate-app-one-eta.vercel.app/login
- Supabaseプロジェクト名：realestate-app

---
> Source: [kurobuchicken-cell/realestate-app](https://github.com/kurobuchicken-cell/realestate-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
