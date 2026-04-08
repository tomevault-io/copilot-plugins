## agent-factory-mcp

> `qwencode-mcp-server` は、Qwen AI モデルを Claude などの AI アシスタントに統合するための Model Context Protocol (MCP) サーバーです。このサーバーはブリッジとして機能し、ユーザーが主要な AI アシスタントを通じて、大規模なコードベースやファイルの分析に長けた Qwen の能力を直接活用できるようにします。

# qwencode-mcp-server

## プロジェクト概要

`qwencode-mcp-server` は、Qwen AI モデルを Claude などの AI アシスタントに統合するための Model Context Protocol (MCP) サーバーです。このサーバーはブリッジとして機能し、ユーザーが主要な AI アシスタントを通じて、大規模なコードベースやファイルの分析に長けた Qwen の能力を直接活用できるようにします。

サーバーは外部の `qwen` CLI をラップしたツールを公開し、プロンプトの転送や `@file` 構文によるファイルコンテキストの解決を処理します。

## アーキテクチャ

このプロジェクトは以下の技術で構成されています。
-   **Node.js**: 実行環境
-   **TypeScript**: 型安全な開発
-   **@modelcontextprotocol/sdk**: MCP サーバー標準の実装
-   **Qwen CLI**: このサーバーが実際の AI 処理を実行するために呼び出す外部依存ツール

**データフロー:**
1.  MCP クライアント（例：Claude）がツール呼び出しリクエスト（例：`ask-qwen`）を送信。
2.  `qwencode-mcp-server` が `stdio` 経由でリクエストを受信。
3.  サーバーが `zod` を使用して引数を検証。
4.  `qwenExecutor` ユーティリティが、外部 `qwen` CLI を呼び出すシェルコマンドを構築。
5.  `qwen` CLI がリクエストを処理（`@` で参照されたローカルファイルを読み込む場合がある）し、結果を返却。
6.  `qwencode-mcp-server` が出力をキャプチャし、MCP クライアントに返却。

## ビルドと実行

### 前提条件
-   Node.js (v16 以上)
-   システムパスにインストール・設定済みの `qwen` CLI ツール

### 主要なコマンド

-   **依存関係のインストール:**
    ```bash
    npm install
    ```
-   **ビルド (TypeScript のコンパイル):**
    ```bash
    npm run build
    ```
    出力は `dist/` ディレクトリに生成されます。
-   **実行 (本番):**
    ```bash
    npm start
    ```
-   **開発モード:**
    ```bash
    npm run dev
    ```
    ビルドしてサーバーを実行します。
-   **テスト:**
    ```bash
    npm test
    ```
-   **静的解析 (Lint):**
    ```bash
    npm run lint
    ```

## 主要なファイル

-   **`src/index.ts`**: アプリケーションのエントリポイント。MCP サーバーを初期化し、ツールやプロンプトのリクエストハンドラを設定し、接続トランスポート (`stdio`) を管理します。
-   **`src/tools/registry.ts`**: ツールの登録と検出を管理します。`UnifiedTool` インターフェースを定義し、内部のツール定義を MCP 準拠のスキーマに変換します。
-   **`src/tools/ask-qwen.tool.ts`**: コアとなる `ask-qwen` ツールの実装。入力（プロンプト、モデル）を定義し、実行を Qwen エグゼキューターに委譲します。
-   **`src/utils/qwenExecutor.ts`**: 外部 `qwen` CLI を呼び出すためのシェルコマンドの構築と実行を担当します。`@` を含むプロンプトを引用符で囲むなどの引数フォーマットを処理します。
-   **`package.json`**: プロジェクトのメタデータ、スクリプト、依存関係を定義します。

## 利用可能なツール

-   **`ask-qwen`**: 主要なツール。
    -   **入力引数**:
        -   `prompt` (string, 必須): Qwen へのクエリ。コンテキスト指定のための `@filename` 構文をサポート。
        -   `model` (string, 任意): 使用する特定の Qwen モデル (例: `qwen-max`)。
    -   **機能**: `qwen -p <prompt> [-m <model>]` を実行します。

## 開発規約

-   **ツールの定義**: 新しいツールは `UnifiedTool` オブジェクト（`src/tools/registry.ts` を参照）として定義し、レジストリに追加してください。これにより、MCP ツールおよびプロンプトの両方として自動的に公開されます。
-   **検証**: ツールの入力スキーマの定義と検証には `zod` を使用してください。
-   **エラー処理**: エラーは適切にキャッチし、サーバーのクラッシュを防ぐために構造化されたエラーレスポンスとして MCP クライアントに返してください。
-   **ロギング**: デバッグやエラー情報には `Logger` ユーティリティを使用してください。

## 使い方

このサーバーは、MCP クライアントの設定ファイル（例：`claude_desktop_config.json`）で設定して使用することを想定しています。

**設定例:**
```json
{
  "mcpServers": {
    "qwen": {
      "command": "node",
      "args": ["/path/to/qwencode-mcp-server/dist/index.js"]
    }
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/utenadev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/utenadev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
