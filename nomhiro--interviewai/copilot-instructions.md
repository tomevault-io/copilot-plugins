## interviewai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

KINTOテクノロジーズの採用PR記事を作成するインタビューAIシステム。3フェーズのワークフロー（企画 → インタビュー → 記事生成）をAzure OpenAIで実行する。バックエンドはFastAPI、フロントエンドはNext.js。

## 開発コマンド

### バックエンド (FastAPI)
```bash
# 依存パッケージのインストール
pip install -r backend/requirements.txt

# 開発サーバー起動（プロジェクトルートから）
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

### フロントエンド (Next.js 16)
```bash
cd frontend
npm install
npm run dev      # 開発サーバー（ポート3000）
npm run build    # プロダクションビルド
npm run lint     # ESLint
```

### 同時起動
バックエンド（ポート8000）とフロントエンド（ポート3000）を別ターミナルで起動する。フロントエンドは `NEXT_PUBLIC_API_BASE`（デフォルト: `http://127.0.0.1:8000/api`）でバックエンドに接続する。

## アーキテクチャ

### 3フェーズワークフロー
1. **企画** (`/api/planning/*`): A2UIによる決定論的5ステップ状態機械でインタビュー構成を設計。LLMはStep 3のテーマ生成でのみ使用。
2. **インタビュー** (`/api/interview/*`): SSEストリーミングチャットでLLMがインタビュアーとして質問。複数人インタビュー対応。リアルタイムでマインドマップを構築。
3. **記事生成** (`/api/generation/*`): 2パス生成 — まずトランスクリプトから引用を抽出し、次にWantedly風Markdownの記事を作成。

### バックエンド (`backend/`)
- **`main.py`**: FastAPIアプリのエントリーポイント。localhost:3000向けCORS設定。ヘルスチェック: `/api/health`。
- **`config.py`**: Pydantic Settingsで`.env`からAzure OpenAI設定を読み込み。
- **`routers/`**: フェーズごとに1ルーター（`planning.py`, `interview.py`, `generation.py`）。
- **`services/`**: ビジネスロジック — `llm_client.py`（非同期Azure OpenAI）, `interview_manager.py`（セッションCRUD）, `planning_state_machine.py`, `article_generator.py`, `a2ui_builder.py`。
- **`prompts/`**: フェーズごとのシステムプロンプトテンプレート + `topic_classifier.py`（マインドマップ用）。
- **`references/`**: 参考データローダーとWantedly記事スタイルテンプレート。

### フロントエンド (`frontend/src/`)
- **`app/`**: Next.js 16 App Router — `page.tsx`（ホーム/セッション一覧）, `planning/page.tsx`, `interview/[id]/page.tsx`, `result/[id]/page.tsx`。
- **`components/`**: `PlanningWizard.tsx`（A2UI）, `interview/`サブフォルダ（InterviewLayout, MindMap*, ResponseInput等）, `ArticlePreview.tsx`, `ThemeToggle.tsx`。
- **`lib/`**: APIクライアント（`api.ts`）, SSEストリームパーサー（`a2ui-stream.ts`, `interview-stream.ts`, `generation-stream.ts`）, TypeScript型定義。
- **`hooks/`**: `useAutoResizeTextarea.ts`。

### データ保存
セッション状態は `data/sessions/<id>/session.json` にJSONファイルとして保存。データベースは不使用。

## 重要な規約

### SSEストリーミング
すべてのLLMインタラクションは`sse-starlette`のServer-Sent Eventsを使用。フロントエンドはネイティブ`fetch` + `ReadableStream`で受信（外部SSEライブラリ不使用）。イベントタイプはフェーズにより異なる: `message`, `a2ui`, `mindmap`, `done`, `progress`。

### A2UI SDK (@a2ui-sdk/react v0.4.0, protocol v0.8)
- Column/Rowの`children`は`{"explicitList": [...]}`形式。プレーン配列は不可。
- Cardは`child`（単数形の文字列）を使用。`children`（配列）ではない。
- `MultipleChoice`コンポーネント（`ChoicePicker`ではない）: propsは`label`, `selections`, `options`, `maxAllowedSelections`。

### 複数人インタビュー
セッションは人物ごとのデータを `session.interviews["0"]`, `session.interviews["1"]` 等に格納。レガシーの1人インタビューセッションは `session.interview_history` を直接使用。

### 参考データ
`video/`に4セットのインタビュー書き起こし（日本語ファイル名）。トランスクリプトは話者ラベルなしの生音声テキスト。記事スタイルはWantedly準拠: `##`見出し、`――`質問、`「」`引用、`>`引用ブロック。

## 環境変数
- バックエンドの`.env`に必要: `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_DEPLOYMENT`, `AZURE_OPENAI_API_VERSION`。
- フロントエンドは `NEXT_PUBLIC_API_BASE` でバックエンドURLを上書き可能。
- `video/`内の日本語ファイルパスはReadツールでエンコーディング問題が発生する場合あり — Serena MCPツール（`mcp__serena__read_file`）を使用すること。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nomhiro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nomhiro)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
