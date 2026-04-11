## barca-fm

> このドキュメントは、Geminiと協力してポッドキャストのエピソードを更新するための手順をまとめたものです。

# ポッドキャストエピソード更新手順

このドキュメントは、Geminiと協力してポッドキャストのエピソードを更新するための手順をまとめたものです。
回答は指定が無い限り日本語でお願いします。

---

## 基本的な更新フロー

1.  **ユーザーからの情報提供**: ユーザーは、新しいエピソードの元ネタをテキストで提供します。
2.  **Geminiによるファイル作成**: 提供された情報に基づき、Geminiは`content/`ディレクトリの最新エピソード番号を確認し、次の番号で新しいMarkdownファイル（例: `52.md`）を作成します。
    *   **Show Notesの構成ルール**: ポッドキャストで話しやすいように、ファイルのメインコンテンツは以下のセクション順で構成してください。
        1.  **試合結果**（直近の試合の詳細）
        2.  **ピッチ外の出来事**（戦術的・心理的な話題、選手・監督の動向など）
        3.  **その他ニュース＆噂**（移籍、インフラ、ライバルチームの動向など）

作業全体の流れについては、 `WORKFLOW.md` を参照してください。

---

## 開発サーバーの起動

ローカルで開発サーバーを起動するには、以下の手順を実行します。

1.  仮想環境をアクティブにします。
    ```bash
    source venv/bin/activate
    ```
2.  開発サーバーを起動します。
    ```bash
    make devserver
    ```
    サーバーは `http://localhost:8000` で利用可能になります。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naga41)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/naga41)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
