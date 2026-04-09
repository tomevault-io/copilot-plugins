## pyproj-template

> - **Runtime**: Python 3.13+, managed with [UV](https://docs.astral.sh/uv/)

# CLAUDE.md

## Tech Stack

- **Runtime**: Python 3.13+, managed with [UV](https://docs.astral.sh/uv/)
- **Database**: DuckDB via duckdb-engine (SQLAlchemy dialect)
- **Data Models**: SQLModel (SQLAlchemy + Pydantic)
- **Data Processing**: pandas + numpy
- **Config**: pydantic-settings
- **CLI**: rich-click
- **Testing**: pytest + factory-boy

## Common Commands

```bash
uv sync --group dev       # 依存関係のインストール
uv run tox -e test        # テスト実行
uv run tox -e lint        # Lint・フォーマット
uv run tox -e typecheck   # 型チェック
uv run tox -e imports     # インポート依存チェック
uv run tox                # 全環境実行
uv run python main.py     # アプリ実行

# marimo ノートブック
uv run marimo edit notebooks/{name}.py   # 編集モード
uv run marimo run  notebooks/{name}.py   # アプリモード（読み取り専用）
```

## Architecture

```
src/
├── core/          # 設定・DB接続（最下層）
├── models/        # SQLModel モデル定義
├── schemas/       # Pydantic スキーマ
├── repos/         # データアクセス層（query.py / session.py が基盤）
├── services/      # 複数リポジトリをまたぐロジック
├── dependencies/  # DB接続ファクトリ
├── middleware/    # ロギング・変換・横断処理
└── usecases/      # ユースケース（最上位層）
```

### 層の依存ルール

```
src.usecases → src.middleware → src.dependencies → src.services
→ src.repos → src.schemas → src.models → src.core
```

各層は自分より下位の層のみインポート可能（import-linter で強制）。

## ヘルパー

| モジュール | 提供する機能 |
|----------|------------|
| `src/core/utils/duckdb.py` | `load_parquet/csv/json/file(conn, path, table_name)` - ファイル→DuckDBテーブル登録 |
| `src/core/utils/path.py` | `data_path(*parts)` - `settings.data_dir` 起点のパス解決 |
| `src/repos/query.py` | `QueryManager.paginate(limit, offset)` - クエリへのページネーション適用 |
| `src/repos/session.py` | `SelectOfScalarSessionManager.to_dataframe()` - クエリ結果→DataFrame変換 |
| `src/repos/session.py` | `SelectOfScalarSessionManager.from_dataframe(df)` - DataFrame→SQLModelリスト変換 |
| `src/middleware/logging.py` | `setup_logging(level)` / `get_logger(name)` - ロギング設定 |

DataFrame のスキーマ検証には **pandera** を使う（`validate_dict.py` の代替）。

## Skills（スラッシュコマンド）

| コマンド | 内容 |
|---------|------|
| `/create-model` | `src/models/{entity}/models.py` の SQLModel モデル作成 |
| `/create-repo` | `src/repos/{entity}.py` のリポジトリ作成 |
| `/create-usecase` | `src/usecases/{entity}/{action}.py` のユースケース作成 |
| `/create-service` | `src/services/{service}.py` のサービス作成 |
| `/create-schema` | `src/schemas/{entity}.py` の Pydantic スキーマ作成 |
| `/create-test` | `tests/` 以下のテストファイル作成 |
| `/create-cli` | `main.py` への CLI コマンド追加 |
| `/create-notebook` | `notebooks/{name}.py` の marimo ノートブック作成 |

## Notebooks

`notebooks/` に Jupyter (`.ipynb`) と marimo (`.py`) を共存させる。
`notebooks/template.py` が marimo のスタートテンプレート。

marimo の特徴:
- 各セルは `@app.cell` デコレータの関数。引数＝依存変数、戻り値＝公開変数（リアクティブ）
- 通常の Python ファイルのためdiff-friendly・import 可能
- `src/repos/{entity}.to_dataframe()` でリポジトリ結果を直接 DataFrame に変換して可視化

## Subagents

| エージェント | 用途 |
|------------|------|
| `data-analyst` | DuckDB SQL・pandas パイプラインの設計・実装 |
| `repo-builder` | model → repo → test をまとめて一括生成 |
| `pipeline-designer` | repo → service → usecase → CLI のフルパイプライン設計 |
| `notebook-designer` | marimo ノートブック（`notebooks/*.py`）をプロジェクトアーキテクチャと統合して作成 |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TsuMakoto)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TsuMakoto)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
