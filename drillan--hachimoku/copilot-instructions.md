## hachimoku

> マルチエージェントコードレビュー CLI ツール。

# hachimoku Development Guidelines

マルチエージェントコードレビュー CLI ツール。
憲法全文: `.specify/memory/constitution.md` (v1.0.0)

## Tech Stack

- Python 3.13+ / pydantic v2 / typer
- エージェント定義: TOML（`.hachimoku/agents/*.toml`）
- 設定: `.hachimoku/config.toml`

## Project Structure

```text
src/hachimoku/    # メインパッケージ
tests/            # テスト（1機能=1ファイル: test_X.py ← X.py）
specs/            # 仕様書（SpecKit: 001-name, 002-name...）
```

<important if="you need to run tests, lint, or quality checks">

## Commands

```bash
uv --directory $PROJECT_ROOT run pytest          # テスト
uv --directory $PROJECT_ROOT run ruff check .    # リント
uv --directory $PROJECT_ROOT run ruff format .   # フォーマット
uv --directory $PROJECT_ROOT run mypy .          # 型チェック
# コミット前必須: ruff check --fix . && ruff format . && mypy .
```

</important>

<important if="you are implementing a new feature or fixing a bug">

## 開発ワークフロー（憲法 Art.1 + Art.2 準拠）

すべての機能実装は、以下の順序に従う。フェーズをスキップしてはならない。

### Phase 1: 仕様（DDD 外部ループ）

1. **仕様書の確認**: `specs/<issue番号>-<name>/spec.md` を読み、実装対象の要件を把握する
2. **仕様書が存在しない場合**: 実装を停止し、ユーザーに仕様定義を提案する
3. **仕様書が存在する場合**: 変更内容に合わせて仕様書を更新する
4. **仕様が曖昧な場合**: 実装を停止し、ユーザーに明確化を要求する
5. **ドキュメント影響確認**: プランモード時は「プランモード必須チェック」に従い Documentation Impact を記載する

### Phase 2: テストと実装（TDD 内部ループ）

仕様が確定した状態で、グローバル CLAUDE.md の TDD 手順に従う:

1. **テスト作成**: 仕様に基づきユニットテストを作成する
2. **ユーザー承認**: テストコードをユーザーに提示し承認を得る
3. **Red 確認**: テストが失敗することを確認する
4. **実装（Green）**: テストが通る最小限の実装を行う
5. **リファクタリング**: テストが通る状態を維持しつつコードを改善する

### Phase 3: 品質チェック

1. `ruff check --fix . && ruff format . && mypy .` を実行する
2. 全エラーを解消するまでコミットしない

</important>

## プロジェクト固有ルール

以下はグローバル CLAUDE.md に**ない**、hachimoku 固有のルール。

<important if="you are modifying CLI commands or output formatting">

### CLI 設計（憲法 Art.3）

- stdout: レビューレポートのみ / stderr: 進捗・ログ・エラー
- 終了コード: 0=成功, 1=Critical, 2=Important, 3=実行エラー, 4=入力エラー
- 全コマンドに `--help` 必須、エラーメッセージに解決方法を含める

</important>

<important if="you are adding or modifying agent definitions">

### データ駆動型アーキテクチャ（憲法 Art.3）

- エージェントは TOML 定義ファイルで外部化（コード変更なしで拡張可能）
- 設定は `.hachimoku/config.toml` で一元管理

</important>

### シンプルさ（憲法 Art.4）

- 最大3プロジェクト構造。追加には文書化された正当な理由が必要
- フレームワーク機能を直接使用。不必要なラッパー禁止

<important if="you are writing or reviewing docstrings">

### Docstring（憲法 Art.10, SHOULD）

- public モジュール・クラス・関数に Google-style docstring 推奨
- 複雑な private 関数（10行以上）にも推奨

</important>

### 命名規則（憲法 Art.11）

- Git ブランチ・コミット: `.claude/git-conventions.md` に従う
- SpecKit ディレクトリ: `<3桁issue番号>-<name>`（例: `002-domain-models`）

<important if="you are modifying files in src/hachimoku/agents/_builtin/">

### ビルトインエージェント TOML の保護

- `src/hachimoku/agents/_builtin/*.toml` は、当該ファイル自体が変更対象である場合にのみ修正する
- リファクタリングや一括置換による巻き込み修正を行わない
- `model` フィールドのプレフィックスは `claudecode:` を使用する（`anthropic:` は不可）

</important>

<!-- MANUAL ADDITIONS START -->

<important if="you are building or updating documentation">

## ドキュメント

- ビルド: `make -C docs html`（全言語ビルド）
- 日本語のみビルド: `make -C docs html-ja`
- 英語のみビルド: `make -C docs html-en`
- ルール: `.claude/docs.md`
- ソースディレクトリ: `docs/ja/`（日本語ドキュメント）、`docs/en/`（英語ドキュメント）
- 共有リソース: `docs/_examples/`, `docs/_static/`, `docs/_templates/`
- `docs/_examples/` は英語で統一（ja/en 共通で literalinclude 参照するため言語別に分離しない）

</important>

<important if="you are creating GitHub issues">

### GitHub Issue 作成ルール

- Issue 作成時は `.github/ISSUE_TEMPLATE/` のテンプレートに従うこと
  - バグ報告: `bug.yml`（概要・再現手順・期待/実際の動作・Review Context）
  - 機能要望: `feature.yml`（概要・受け入れ基準・Review Context）
- Review Context は分かる範囲で記入する。各フィールドの記入指針:
  - 関連仕様: spec ファイルパス + セクション/要件番号、関連 Issue/PR
  - 影響範囲: diff 外で影響を受けるファイルパス（ファイル単位）+ 影響の種類
  - 既存パターン参照: 参照先ファイルパス + 具体的なパターン名
- 空欄は許容される。不正確な情報よりも空欄の方が望ましい
- 詳細ガイドライン: `docs/ja/review-context.md`
- `gh issue create` 使用時は `--body` でテンプレートの全セクションを含める

</important>

<important if="you are entering plan mode or creating implementation plans">

### プランモード必須チェック（憲法 Art.2 準拠）

Plan モード（EnterPlanMode / `/speckit.plan` / `/start-issue`）で計画を作成する際、
ExitPlanMode の前に以下のドキュメント影響確認を必ず実施すること:

1. **specs/**: 変更対象機能の spec.md / data-model.md に更新が必要か確認
2. **docs/**: Sphinx ドキュメント（docs/ 配下）に反映すべき内容があるか確認
3. **README.md**: ユーザー向け情報（使い方、設定例、コマンド等）に変更が必要か確認

確認結果はプランに **Documentation Impact** セクションとして含めること:

```markdown
## Documentation Impact

| 対象 | 更新要否 | 内容 |
|------|---------|------|
| specs/NNN-xxx/spec.md | 要/否 | （要の場合: 概要） |
| docs/*.md | 要/否 | （要の場合: 概要） |
| README.md | 要/否 | （要の場合: 概要） |
```

影響がない場合も「影響なし」と明記すること。

</important>

<!-- MANUAL ADDITIONS END -->

## Active Technologies
- Python 3.13+ + pydantic ≥2.12.5（バリデーション）, tomllib（Python 標準ライブラリ、TOML パース）, importlib.resources（ビルトイン定義のパッケージ内配置）, fnmatch（ファイルパターンマッチング）, re（コンテンツパターンマッチング） (003-agent-definition)
- ファイルシステム（TOML 定義ファイル） (003-agent-definition)
- Python 3.13+ + pydantic ≥2.12.5（バリデーション）, tomllib（Python 標準ライブラリ、TOML パース）, pathlib（ファイルシステム操作） (004-configuration)
- ファイルシステム（TOML 設定ファイル） (004-configuration)
- Python 3.13+ + pydantic-ai（エージェント実行基盤）, pydantic ≥2.12.5（バリデーション）, asyncio（並列実行・タイムアウト制御） (005-review-engine)
- N/A（ファイルシステムは設定・エージェント定義の読み込みのみ、004/003 が担当済み） (005-review-engine)
- Python 3.13+ + Typer ≥0.21.1（CLI フレームワーク）, pydantic ≥2.12.5（バリデーション）, pydantic-ai ≥1.56.0（エンジン経由） (006-cli-interface)
- ファイルシステム（`.hachimoku/` ディレクトリ構造の初期化） (006-cli-interface)

## Recent Changes
- 003-agent-definition: Added Python 3.13+ + pydantic ≥2.12.5（バリデーション）, tomllib（Python 標準ライブラリ、TOML パース）, importlib.resources（ビルトイン定義のパッケージ内配置）, fnmatch（ファイルパターンマッチング）, re（コンテンツパターンマッチング）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drillan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
