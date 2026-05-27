## fastmcp

> FastMCP の基礎知識

## FastMCPとは

FastMCPは、Model Context Protocol (MCP) サーバーを簡単に構築するためのPythonフレームワークです。Pythonの関数をデコレータで装飾するだけで、MCPサーバーを素早く構築できます。

> 🎉 FastMCPは現在、公式のMCP SDKに統合されています！
> 公式のModel Context Protocol Python SDK: [github.com/modelcontextprotocol/python-sdk](mdc:https:/github.com/modelcontextprotocol/python-sdk)

## 主な特徴

- **高速**: 高レベルなインターフェースにより、少ないコードで迅速な開発が可能
- **シンプル**: 最小限のボイラープレートコードでMCPサーバーを構築
- **Pythonic**: Python開発者にとって自然な感覚で使用可能
- **完全**: MCPの仕様を完全に実装することを目指している

## インストール方法

FastMCPは `uv` を使ってインストールすることが推奨されています：

```bash
uv pip install fastmcp
```

macOSでは、Claude Desktopアプリで利用するために、Homebrewを使って `uv` をインストールする必要がある場合があります：

```bash
brew install uv
```

あるいは、デプロイせずにSDKを使用する場合は、pipを使用することもできます：

```bash
pip install fastmcp
```

## クイックスタート

以下は、簡単な計算ツールを提供するMCPサーバーの例です：

```python
# server.py
from fastmcp import FastMCP

# MCPサーバーの作成
mcp = FastMCP("Demo")

# 足し算ツールの追加
@mcp.tool()
def add(a: int, b: int) -> int:
    """二つの数値を足し算する"""
    return a + b

# サーバーの実行
if __name__ == "__main__":
    mcp.run()
```

これだけで、動作するMCPサーバーが完成します！

## 主要コンポーネント

FastMCPは以下の主要コンポーネントをサポートしています：

### 1. サーバー

サーバーはFastMCPのメインオブジェクトで、すべての機能の中心となります：

```python
from fastmcp import FastMCP

mcp = FastMCP(
    "サーバー名",
    host="0.0.0.0",  # オプション
    port=52229,      # オプション
)
```

### 2. ツール

ツールはLLMが呼び出せる関数です。デコレータを使って簡単に定義できます：

```python
@mcp.tool()
def my_tool(param1: str, param2: int) -> str:
    """
    ツールの説明
    
    Args:
        param1: 最初のパラメータの説明
        param2: 2番目のパラメータの説明
        
    Returns:
        結果の説明
    """
    # ツールの実装
    return f"結果: {param1}, {param2}"
```

### 3. リソース

リソースはLLMがアクセスできるデータです：

```python
@mcp.resource("echo://{message}")
def echo_resource(message: str) -> str:
    """リソースとしてメッセージをエコーする"""
    return f"Resource echo: {message}"
```

### 4. プロンプト

プロンプトはLLMに特定のタスクを実行させるための指示です：

```python
@mcp.prompt()
def analyze_table(table: str) -> str:
    """テーブル分析用のプロンプトテンプレートを作成する"""
    return f"""このデータベーステーブルを分析してください:
テーブル: {table}
スキーマ: 
{get_schema()}

構造と関係性についてどのような洞察を提供できますか？"""
```

## サーバーの実行方法

FastMCPサーバーを実行するには、いくつかの方法があります：

### 1. 開発モード（推奨）

開発とテスト用に、以下のコマンドを使用します：

```bash
fastmcp dev server.py
```

### 2. Claude Desktop統合（通常使用）

Claude Desktopで使用するためにサーバーをインストールします：

```bash
fastmcp install server.py
```

### 3. 直接実行（高度なユースケース）

サーバーを直接実行することもできます：

```python
if __name__ == "__main__":
    mcp.run(transport="stdio")  # または "sse"
```

## 実装例

### エコーサーバー

リソース、ツール、プロンプトを示す簡単なサーバー：

```python
from fastmcp import FastMCP

mcp = FastMCP("Echo")

@mcp.resource("echo://{message}")
def echo_resource(message: str) -> str:
    """リソースとしてメッセージをエコーする"""
    return f"Resource echo: {message}"

@mcp.tool()
def echo_tool(message: str) -> str:
    """ツールとしてメッセージをエコーする"""
    return f"Tool echo: {message}"

@mcp.prompt()
def echo_prompt(message: str) -> str:
    """エコープロンプトを作成する"""
    return f"このメッセージを処理してください: {message}"
```

### SQLiteエクスプローラー

データベース統合を示すより複雑な例：

```python
from fastmcp import FastMCP
import sqlite3

mcp = FastMCP("SQLite Explorer")

@mcp.resource("schema://main")
def get_schema() -> str:
    """リソースとしてデータベーススキーマを提供する"""
    conn = sqlite3.connect("database.db")
    schema = conn.execute(
        "SELECT sql FROM sqlite_master WHERE type='table'"
    ).fetchall()
    return "\n".join(sql[0] for sql in schema if sql[0])

@mcp.tool()
def query_data(sql: str) -> str:
    """SQLクエリを安全に実行する"""
    conn = sqlite3.connect("database.db")
    try:
        result = conn.execute(sql).fetchall()
        return "\n".join(str(row) for row in result)
    except Exception as e:
        return f"Error: {str(e)}"
```

## TypeScript版 FastMCP

Python版のFastMCPに加えて、TypeScript版のFastMCPも存在します。これは同様の機能を提供しますが、TypeScript/JavaScript環境向けに設計されています。

### 特徴

- シンプルなツール、リソース、プロンプト定義
- 認証
- セッション管理
- 画像コンテンツ
- ロギング
- エラーハンドリング
- SSE（Server-Sent Events）
- CORS（デフォルトで有効）
- 進捗通知
- 型付きサーバーイベント

### インストール

```bash
npm install fastmcp
```

### 基本的な使用方法

```typescript
import { FastMCP } from "fastmcp";
import { z } from "zod";

const server = new FastMCP({
  name: "My Server",
  version: "1.0.0",
});

server.addTool({
  name: "add",
  description: "Add two numbers",
  parameters: z.object({
    a: z.number(),
    b: z.number(),
  }),
  execute: async (args) => {
    return String(args.a + args.b);
  },
});

server.start({
  transportType: "stdio",
});
```

### テスト方法

```bash
npx fastmcp dev server.js
# または
npx fastmcp dev server.ts
```

## 参考リンク

- [Python FastMCP GitHub](mdc:https:/github.com/jlowin/fastmcp)
- [TypeScript FastMCP GitHub](mdc:https:/github.com/punkpeye/fastmcp)
- [公式MCP Python SDK](mdc:https:/github.com/modelcontextprotocol/python-sdk)

---
> Source: [tumf/grafana-loki-mcp](https://github.com/tumf/grafana-loki-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
