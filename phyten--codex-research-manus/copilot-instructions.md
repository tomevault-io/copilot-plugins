## codex-research-manus

> このファイルは、OpenAI Codexがこのリポジトリを理解し、操作するための指示を記述します。

# AGENTS.md

このファイルは、OpenAI Codexがこのリポジトリを理解し、操作するための指示を記述します。

## リポジトリの概要

このリポジトリは、書籍「Codex Research Manus」の原稿を管理するためのものです。
各章は `chapters` ディレクトリ内にMarkdownファイルとして格納されています。

## 主な目的

このリポジトリの主な目的は、Markdownで記述された原稿からPDFを自動生成することです。
GitHub Actionsを利用して、リポジトリへのプッシュをトリガーにPDFビルドプロセスを実行します。

## Codexへの指示

1.  **PDFビルド**:
    *   `chapters` ディレクトリ内のMarkdownファイルを結合し、Pandocなどのツールを使用してPDFに変換するGitHub Actionsワークフローを作成してください。
    *   Markdownファイルはファイル名の昇順で結合してください（例: `00_document_structure.md`, `01_introduction.md`, ...）。
    *   生成されるPDFには目次を含めてください。
    *   数式やコードブロックが正しくレンダリングされるように設定してください。
    *   必要に応じて、日本語フォントの設定も考慮してください。
2.  **依存関係の管理**:
    *   PDFビルドに必要なツール（Pandoc、LaTeXなど）やフォントをGitHub Actionsのワークフロー内でセットアップするようにしてください。
    *   もし特定のバージョンが必要な場合は、それを指定してください。
3.  **出力**:
    *   生成されたPDFは、GitHub Actionsの成果物としてアップロードされるようにしてください。
    *   可能であれば、リリース作成時に自動的にPDFが添付されるような仕組みも検討してください。
4.  **その他**:
    *   リポジトリのルートに `README.md` がありますが、これは一般的な情報提供用です。Codexへの指示は、この `AGENTS.md` を優先してください。
    *   ファイル操作を行う際は、既存のMarkdownファイルを誤って上書きしないように注意してください。

## 将来的な展望

*   EPUBやHTMLなど、他のフォーマットへの変換も検討しています。
*   カバー画像の追加や、より詳細なスタイリングのカスタマイズも行う予定です。

ご協力ありがとうございます。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phyten)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/phyten)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
