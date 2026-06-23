## fetch-x-posts

> - このリポジトリは `fetch-x-posts` 用です。指定したハッシュタグとJST日時範囲に一致するXポストを、xAI Grok API の `x_search` を使って取得するPython CLIを作ります。

# AGENTS.md

## プロジェクト範囲

- このリポジトリは `fetch-x-posts` 用です。指定したハッシュタグとJST日時範囲に一致するXポストを、xAI Grok API の `x_search` を使って取得するPython CLIを作ります。
- 実装要件は [docs/plan.md](docs/plan.md) を正としてください。
- 現在の [fetch_x_posts.py](fetch_x_posts.py) は試作版です。`docs/plan.md` に沿って再実装する際は、破棄または全面的に置き換えて構いません。

## 実装ルール

- APIキーや秘密情報をコードに直書きしないでください。
- xAI APIキーは `XAI_API_KEY` 環境変数から読み込んでください。
- ユーザーが明示的に依頼するまで、実APIを呼び出す実行や検証は行わないでください。
- API呼び出し部分と、ローカルでテストできるロジックは分離してください。
- xAI SDK呼び出しは `xai_sdk.chat.user` と `xai_sdk.tools.x_search` を使う前提です。
- 日時範囲は検索クエリ内の `since:` / `until:` ではなく、`x_search(from_date=..., to_date=...)` のツール引数に渡してください。
- 日本語ハッシュタグは、[docs/plan.md](docs/plan.md) のファイル名ルールに従って生成ファイル名に残してください。
- プロジェクトで仮想環境用コマンドを定義するまでは、例や確認コマンドでは `python3` を使ってください。

## 確認

- Pythonコードを編集した後は、最低限以下を実行してください。

```bash
python3 -m py_compile fetch_x_posts.py
```

- テストを追加した場合は、完了報告前に該当するテストコマンドを実行してください。現状は以下を使います。

```bash
python3 -m unittest tests/test_fetch_x_posts.py
```

## コミットルール

- ユーザーにコミットを依頼された場合のみコミットしてください。
- コミット前に `git status --short` と `git diff --stat` で対象差分を確認してください。
- Pythonコードを含むコミットでは、最低限 `python3 -m py_compile fetch_x_posts.py` と `python3 -m unittest tests/test_fetch_x_posts.py` を実行してください。
- コミットメッセージは日本語で書いてください。
- 件名は Conventional Commits 風に `type: 要約` の形式にしてください。
- 主な `type` は以下を使ってください。
  - `feat`: 機能追加
  - `fix`: 不具合修正
  - `docs`: ドキュメントのみの変更
  - `test`: テスト追加・修正
  - `refactor`: 振る舞いを変えない整理
  - `chore`: 開発環境や雑務
- 件名は短く具体的にしてください。例: `feat: Xポスト取得CLIを実装`
- 本文には、何を変更したかと、必要なら理由や確認内容を簡潔に書いてください。
- 無関係な変更を同じコミットに混ぜないでください。

---
> Source: [tenkao/fetch-x-posts](https://github.com/tenkao/fetch-x-posts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
