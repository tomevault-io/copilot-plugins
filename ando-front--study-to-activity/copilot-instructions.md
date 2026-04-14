## study-to-activity

> このファイルは「Study to Activity」プロジェクトに関するコンテキストを提供し、AIアシスタントがプロジェクトの構造、目的、規約を理解するのを助けるために設計されています。

# GEMINI.md - プロジェクトコンテキスト

このファイルは「Study to Activity」プロジェクトに関するコンテキストを提供し、AIアシスタントがプロジェクトの構造、目的、規約を理解するのを助けるために設計されています。

## プロジェクト概要

**Study to Activity (S2A)** は、子供の学習進捗を管理し、Nintendo Switchやタブレットでの遊び時間といったアクティビティ時間で報酬を与えるために設計されたWebアプリケーションです。

- **ビジョン:** 子供たちの学習努力を具体的な報酬に変えることでモチベーションを高め、同時に親にはスクリーンタイムを明確かつ自動的に管理する方法を提供します。
- **コア機能:**
    1. 親が学習計画やタスクを設定します。
    2. 子供がタスクを完了し、進捗を記録します。
    3. 親が完了したタスクを承認します。
    4. 事前に定義されたルールに基づき、システムが自動的に「アクティビティ時間」をデジタル「ウォレット」に付与します。
    5. このウォレットの残高は、（将来的には）デバイスの利用時間を制御するために使用できます。

## アーキテクチャと技術

これは、**FastAPI** フレームワークを使用したPythonベースのバックエンドプロジェクトです。

- **バックエンド:**
    - **フレームワーク:** FastAPI
    - **データベース:** ORMとしてSQLAlchemyを使用します。
        - **ローカル開発:** デフォルトでは、プロジェクトルートに `s2a.db` という名前の **SQLite** ファイルが自動的に作成されます。
        - **本番環境:** `DATABASE_URL` 環境変数を設定することで、**PostgreSQL** などの他のデータベースに接続できます。`psycopg2-binary` が `backend/requirements.txt` に含まれているのはこのためです。
    - **Webサーバー:** Uvicorn
    - **依存関係:** Pythonのすべての依存関係は `backend/requirements.txt` に記載されています。
- **フロントエンド (計画中):**
    - `docs/prd.md` と `backend/main.py` のCORS設定から、このAPIを利用する別のフロントエンド（おそらく **Next.js (React)** アプリケーション）が存在することを示唆しています。

### 主要なディレクトリとファイル

- `backend/`: すべてのバックエンドソースコードを含みます。
    - `main.py`: メインのFastAPIアプリケーションエントリーポイント。アプリ、ミドルウェア、ルーターを初期化します。
    - `database.py`: SQLAlchemyエンジンとデータベース接続を設定します。
    - `models.py`: SQLAlchemyのデータベースモデル（例: User, StudyPlan, RewardRule）を定義します。
    - `schemas.py`: APIリクエストとレスポンスで使用されるPydanticのデータ検証スキーマを定義します。
    - `routers/`: 関心事（例: `auth.py`, `plans.py`, `tasks.py`）によって論理的に分離された、さまざまなAPIエンドポイントを含みます。
- `docs/`: プロジェクトのドキュメンテーションを含みます。
    - `prd.md`: プロジェクトのビジョン、機能、技術計画の詳細な概要を提供する製品要求仕様書（Product Requirements Document）。

## ビルドと実行

### 1. 依存関係のインストール

開発環境をセットアップするために、必要なPythonパッケージをインストールします。

```bash
pip install -r backend/requirements.txt
```

### 2. バックエンドサーバーの実行

アプリケーションは `uvicorn` を使用して実行します。プロジェクトのルートディレクトリから、以下を実行します。

```bash
uvicorn backend.main:app --reload
```

- サーバーは `http://127.0.0.1:8000` で利用可能になります。
- `--reload` フラグは、コードの変更が検出されたときに自動リロードを有効にします。
- APIドキュメンテーション（Swagger UI）は `http://127.0.0.1:8000/docs` で利用可能です。

## 開発規約

- **モジュール化されたルーター:** APIエンドポイントは `backend/routers/` ディレクトリ内の別々のファイルに整理されています。各ファイルは `fastapi.APIRouter` インスタンスを使用します。
- **関心の分離:** プロジェクトはデータベースモデル（`models.py`）、データスキーマ（`schemas.py`）、APIロジック（ルーター）を分離しています。
- **データベースの初期化:** `main.py` 内の `Base.metadata.create_all(bind=engine)` コマンドは、アプリケーション起動時に定義されたモデルに基づいてデータベーステーブルを自動的に作成します。
- **CORSの有効化:** Next.jsやCreate React Appのような多くのフロントエンド開発サーバーのデフォルトである `localhost:3000` からのリクエストを許可するために、クロスオリジンリソース共有（CORS）が有効化されています。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ando-front)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ando-front)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
