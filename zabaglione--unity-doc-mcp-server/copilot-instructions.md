## unity-doc-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Unity Documentation MCP Server

## Project Overview

Unity公式ドキュメント（Unity 6）をModel Context Protocol経由で提供するMCPサーバー。Unity開発時にClaude Codeが公式ドキュメントを参照して正確な情報を提供できるようにする。

## Core Requirements

### MVP機能
- Unity 6 (6000.x) 公式ZIPドキュメントのダウンロード・解析
- 基本的なテキスト検索機能
- MCPプロトコル経由でのドキュメント検索
- シンプルなキャッシュ機構

### 非機能要件
- 検索レスポンス: 500ms以下
- メモリ使用量: 512MB以下
- 完全なオフライン動作

## Technical Architecture

### 技術スタック
- **Runtime**: Node.js 18+ / TypeScript
- **Database**: SQLite with FTS5（全文検索）
- **Cache**: 簡易メモリキャッシュ
- **Documentation**: Unity公式ZIPファイル

### ディレクトリ構成
```
unity-doc-mcp-server/
├── src/
│   ├── server.ts         # MCPサーバーメイン
│   ├── downloader.ts     # Unity ZIPダウンロード
│   ├── parser.ts         # HTMLパーサー
│   ├── database.ts       # SQLiteデータベース
│   └── search.ts         # 検索エンジン
├── data/
│   ├── unity-zips/       # ダウンロードしたZIP
│   ├── extracted/        # 展開したHTML
│   └── unity.db         # SQLiteデータベース
└── tests/
    └── *.test.ts        # テストファイル
```

## Claude Codeを使った実装計画（1週間）

### Day 1-2: 基本構造とZIPダウンロード
```typescript
// 1. Unity ZIPダウンローダー
class UnityZipDownloader {
  async download(version: string): Promise<void> {
    const url = `https://cloudmedia-docs.unity3d.com/docscloudstorage/en/${version}/UnityDocumentation.zip`;
    // ダウンロードと展開
  }
}

// 2. データベース初期化
interface Document {
  id: string;
  title: string;
  type: 'manual' | 'script-reference';
  content: string;
  html: string;
}
```

### Day 3-4: HTMLパース＆データベース格納
```typescript
// HTMLパーサー
class UnityHtmlParser {
  parse(htmlContent: string): Document {
    // cheerioでHTMLをパース
    // タイトルとコンテンツを抽出
  }
}

// SQLiteスキーマ
CREATE TABLE documents (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  type TEXT NOT NULL,
  content TEXT NOT NULL,
  html TEXT NOT NULL
);

CREATE VIRTUAL TABLE documents_fts USING fts5(
  title, content,
  content=documents,
  tokenize='porter ascii'
);
```

### Day 5-6: MCPサーバー実装
```typescript
// MCPツール定義
const searchUnityDocsTool = {
  name: "search_unity_docs",
  description: "Search Unity 6 documentation",
  inputSchema: {
    type: "object",
    properties: {
      query: {
        type: "string",
        description: "Search query"
      },
      limit: {
        type: "number",
        default: 10
      }
    },
    required: ["query"]
  }
};

// 検索実装
class UnityDocsServer {
  async handleToolCall(name: string, args: any) {
    if (name === "search_unity_docs") {
      const results = await this.search(args.query, args.limit);
      return {
        content: [{
          type: "text",
          text: this.formatResults(results)
        }]
      };
    }
  }
}
```

### Day 7: テスト＆デプロイ
- 基本的な検索テスト
- エラーハンドリング
- README作成

## 実装時の注意点

### コーディング規約
- TypeScript strict mode
- エラーハンドリングを確実に実装
- 外部APIへの依存を避ける

### よく使うコマンド
```bash
# 開発環境のセットアップ
npm install
npm run dev

# テスト実行
npm test
npm run test:watch

# ビルド
npm run build

# Unity ドキュメントのダウンロード
npm run download-docs 6000.1
```

## Quick Start

```bash
# 1. セットアップ
git clone [repository]
npm install

# 2. Unity ドキュメントをダウンロード
npm run download-docs 6000.1

# 3. 開発サーバー起動
npm run dev
```

## MCP設定

```json
{
  "mcpServers": {
    "unity-docs": {
      "command": "node",
      "args": ["dist/server.js"]
    }
  }
}
```

## 大規模コンテキスト解析について

Unity公式ドキュメントは非常に大規模なため、全体の構造把握や複数ファイルの解析が必要な場合は、`USE_GEMINI.md`を参照してGemini CLIを使用してください。

---

**開発期間**: 1週間（Claude Code使用）
**主な機能**: Unity 6ドキュメントの検索・参照

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zabaglione)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zabaglione)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
