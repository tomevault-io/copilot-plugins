## chat-litellm-agent

> このプロジェクトは、**カスタムLiteLLMエージェントラッパー**であり、言語モデルに自律的なツール呼び出し機能を追加します。LiteLLMのプロキシサーバーを通じてLLMリクエストをインターセプトし、ツール実行を自動化することで、クライアントアプリケーションがツール実行ロジックを処理する必要なく、任意の互換性のあるLLMでツールを使用できるようにします。

# chat-litellm-agent リポジトリ分析

## 概要

このプロジェクトは、**カスタムLiteLLMエージェントラッパー**であり、言語モデルに自律的なツール呼び出し機能を追加します。LiteLLMのプロキシサーバーを通じてLLMリクエストをインターセプトし、ツール実行を自動化することで、クライアントアプリケーションがツール実行ロジックを処理する必要なく、任意の互換性のあるLLMでツールを使用できるようにします。

## プロジェクト構造

```
/home/kubo/Documents/chat-litellm-agent/
├── litellm/
│   ├── agent.py           # ツール呼び出し機能を持つカスタムLLM実装
│   └── config.yaml        # LiteLLMプロキシ設定
├── main.py                # シンプルなテストクライアント
├── test_focused.py        # 包括的なテストスイート
├── pyproject.toml         # プロジェクトメタデータと依存関係
├── setup.sh               # インストールスクリプト
├── README.md              # セットアップと使用方法のドキュメント
├── NOTE.md                # 設計ノートと考察
├── .gitignore            # Git除外ルール
├── .python-version       # Pythonバージョン (3.12)
├── uv.lock               # UVパッケージマネージャーロックファイル
└── .webui_secret_key     # Open WebUIシークレットキー
```

## 主要機能

1. **自動ツール実行**: クライアントの介入なしにツールを透過的に実行
2. **ストリーミングサポート**: ツール呼び出しを含む完全なストリーミングサポート
3. **マルチモデルサポート**: ローカル（Ollama）とクラウド（OpenAI）の両方のモデルに対応
4. **OpenAI互換性**: 標準のOpenAI APIフォーマットを使用
5. **WebUI統合**: Open WebUIとシームレスに連携
6. **拡張可能なツールシステム**: `available_functions`レジストリに新しいツールを簡単に追加可能
7. **包括的なロギング**: エージェントの動作をデバッグするための詳細なロギング
8. **非同期ファーストデザイン**: より良いパフォーマンスのための完全非同期実装

## コアコンポーネント

### 1. [litellm/agent.py](litellm/agent.py) (346行)

エージェントの心臓部となるカスタムLLMハンドラー。

**主要クラス:**
- `MyCustomLLM`: `litellm.CustomLLM`を拡張してLLM呼び出しをインターセプト・拡張

**主要メソッド:**
1. `completion()` - 自動ツール実行機能付き同期補完
2. `acompletion()` - 自動ツール実行機能付き非同期補完
3. `astreaming()` - ツール実行サポート付き非同期ストリーミング

**ツール実装:**
- `get_current_weather()`: JSONフォーマットで天気データを返すモックツール
- `tools`: OpenAI互換の関数呼び出しスキーマ定義
- `available_functions`: 関数名を実装にマッピングするレジストリ

**ワークフローパターン（全3メソッド共通）:**
1. クライアントからリクエストを受信
2. ツールスキーマ付きで基盤LLMを呼び出し
3. LLMがツールを呼び出したいか確認（finish_reason == "tool_calls"）
4. ツール呼び出しが必要な場合:
   - ツール関数を実行
   - ツール結果をメッセージ履歴に追加
   - ツール結果を含めて最終LLM呼び出し
5. 最終レスポンスを返す

**ストリーミング固有のロジック:**
- 最初に非ストリーミング呼び出しでツール使用を検出
- ツールが必要な場合は先に実行
- その後、ツール結果を組み込んだ最終レスポンスをストリーミング
- LiteLLM互換性のため`GenericStreamingChunk`フォーマットに変換

### 2. [litellm/config.yaml](litellm/config.yaml)

LiteLLMプロキシ設定ファイル。

**モデル定義:**
- `my-custom-qwen3-0.6b`: Ollamaのqwen3:0.6bをラップするカスタムエージェント
- `my-custom-gpt-5-nano`: OpenAIのgpt-5-nanoをラップするカスタムエージェント
- `qwen3-0.6b`: Ollamaへの直接アクセス（エージェントなし）
- `gpt-5-nano`: OpenAIへの直接アクセス（エージェントなし）

**カスタムプロバイダー設定:**
```yaml
litellm_settings:
  custom_provider_map:
    - provider: "my-custom-llm"
      custom_handler: agent.my_custom_llm
```
これにより、`my-custom-llm/`プレフィックス付きモデルがツール呼び出しエージェントを使用するように登録されます。

**セキュリティ:**
- マスターAPIキー: `sk-1234` (デモ用)

### 3. [main.py](main.py) (28行)

以下を実証するシンプルなテストクライアント:
- `http://localhost:4000`のローカルLiteLLMプロキシへの接続
- OpenAI SDKを使用したプロキシとの通信
- "my-custom-model"エンドポイントのテスト

### 4. [test_focused.py](test_focused.py) (194行)

4つのテストシナリオを含む包括的なテストスイート:

1. **同期天気テスト**: 非ストリーミングのツール呼び出し
2. **ストリーミング天気テスト**: ツール呼び出しを含むストリーミング
3. **通常の会話**: ツールを使用しない会話の検証
4. **複数リクエスト**: 異なる都市への連続的なツール呼び出し

**テストアプローチ:**
- 同期（`OpenAI`）と非同期（`AsyncOpenAI`）両方のクライアントを使用
- レスポンスに期待されるデータが含まれているか確認してツール実行を検証
- 絵文字とステータスメッセージで明確な視覚的フィードバックを提供
- 各シナリオの合格/不合格を示すテストサマリーを生成

## 使用技術とフレームワーク

### コア依存関係 ([pyproject.toml](pyproject.toml)より):
- **LiteLLM** (>=1.79.1 with proxy): ユニバーサルLLMプロキシとゲートウェイ
- **OpenAI SDK** (>=2.7.1): テストとAPI通信用のクライアントライブラリ

### ランタイム環境:
- **Python 3.12**: 言語バージョン
- **UV**: モダンなPythonパッケージマネージャー（pip/poetryの代替）
- **Ollama**: qwen3:0.6bモデル用のローカルLLMランタイム
- **Open WebUI**: Webベースのチャットインターフェース

### サポート技術:
uv.lockの依存関係分析より:
- **aiohttp**: 非同期HTTPクライアント/サーバーフレームワーク
- **asyncio**: 非同期I/Oサポート
- **Pydantic**: データ検証（LiteLLM内部で使用）
- **FastAPI**: Webフレームワーク（LiteLLMプロキシ内部で使用）
- **Starlette**: ASGIフレームワーク（FastAPIの依存関係）

## アーキテクチャとデータフロー

### リクエストフロー:

```
ユーザー (Open WebUI)
    ↓ HTTP POST /chat/completions
LiteLLMプロキシサーバー (:4000)
    ↓ モデル名に基づいてルーティング
MyCustomLLM.astreaming()
    ↓ ツールの必要性を初期チェック
基盤LLM (Ollama/OpenAI)
    ↓ 必要に応じてtool_callsを返す
ツール実行 (get_current_weather)
    ↓ 結果を会話履歴に追加
ツール結果を含む最終LLM呼び出し
    ↓ レスポンスをストリーミング
GenericStreamingChunk変換
    ↓ OpenAI互換フォーマット
ユーザーが最終回答を受信
```

### エージェントループパターン:

エージェントはシンプルかつ効果的なエージェントループを実装しています:
1. **知覚**: ユーザークエリを受信
2. **判断**: LLMがツールが必要かどうかを判断
3. **行動**: ツール関数を実行
4. **反映**: LLMがツール結果を解釈
5. **応答**: 最終回答をユーザーにストリーミング

このパターンは、同期/非同期およびストリーミング/非ストリーミングのわずかなバリエーションを含め、すべての3つのメソッド（completion、acompletion、astreaming）で実装されています。

## エントリーポイント

### プライマリエントリーポイント:
```bash
uv run litellm --config litellm/config.yaml --port 4000
```
LiteLLMプロキシサーバーを起動し、以下を実行します:
- `litellm/config.yaml`から設定をロード
- `agent.my_custom_llm`をインポートして登録
- ポート4000でOpenAI互換APIを公開
- 適切なモデル/ハンドラーにリクエストをルーティング

### モジュールインポートチェーン:
1. LiteLLMプロキシ起動 → config.yamlを読み込み
2. configが`agent.my_custom_llm`を参照
3. Pythonが`/home/kubo/Documents/chat-litellm-agent/litellm/agent.py`をインポート
4. `my_custom_llm`インスタンスがカスタムプロバイダーとして登録
5. `my-custom-llm/*`モデルへのリクエストがこのハンドラーにルーティング

### テストエントリーポイント:
```bash
# シンプルなテスト
uv run python main.py

# 包括的なテストスイート
uv run python test_focused.py
```

### Webインターフェースエントリーポイント:
```bash
DATA_DIR=~/.open-webui uvx --python 3.11 open-webui@latest serve
```

## セットアップと使用ワークフロー

### 初回セットアップ:
```bash
# Ollamaをインストール
curl -fsSL https://ollama.com/install.sh | sh

# UVパッケージマネージャーをインストール
curl -LsSf https://astral.sh/uv/install.sh | sh

# モデルをプル
ollama pull qwen3:0.6b
```

### システムの実行:
```bash
# ターミナル1: Ollamaを起動（まだ実行されていない場合）
# ターミナル2: LiteLLMプロキシを起動
uv run litellm --config litellm/config.yaml --port 4000

# ターミナル3: Open WebUIを起動
DATA_DIR=~/.open-webui uvx --python 3.11 open-webui@latest serve

# ターミナル4: テストを実行
uv run python test_focused.py
```

### cURLでのテスト:
```bash
# モデルリストのテスト
curl -s http://localhost:4000/v1/models -H "Authorization: Bearer sk-1234"

# チャット補完のテスト
curl -X POST 'http://0.0.0.0:4000/chat/completions' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer sk-1234' \
  -d '{"model": "gpt-5-nano", "messages": [...]}'
```

## 設計上の考察 ([NOTE.md](NOTE.md)より)

開発者が文書化した主要な洞察:

1. **Langchain統合**: LangGraph + Open WebUI統合のガイドへのリンク
2. **LiteLLMエージェントのトレードオフ**:
   - 利点: 別のFastAPIサーバーを構築する必要がない
   - 欠点: Function/Pipeline統合には理想的ではない
   - 回避策: UIフィードバックのためにツール完了メッセージをストリーミング可能
   - シンプルさ: エージェントループの実装が簡単
   - 将来: ユーザーの意図を理解するための「トリアージエージェント」が必要になる可能性

## 強みと制限

### 強み:
- シンプルで焦点を絞った実装（346行の単一エージェントファイル）
- 包括的なテストスイートでよくテストされている
- 関心事の明確な分離（config、agent、tests）
- 本番環境対応のロギング
- 同期と非同期の両方をサポート
- ストリーミング互換のツール実行

### 制限:
- サンプルツールが1つのみ（天気）- 拡張が必要
- ツール失敗時のエラーリカバリーなし
- マルチステップの推論/計画なし
- ツール結果の検証なし
- 固定マスターキー（本番環境非対応）
- シングルターンのツール使用に限定（反復的なツール呼び出しなし）

## まとめ

これは、LiteLLMのカスタムハンドラーシステムを通じて任意のLLMに自律的なツール呼び出し機能を追加する方法を実証する、よく設計された**概念実証的エージェントLLMプロキシ**です。このプロジェクトは、シンプルなチャットインターフェースと機能的なAIエージェントの間のギャップを成功裏に埋め、Open WebUIまたは任意のOpenAI互換クライアントを通じてツール対応LLMを簡単にデプロイできるようにします。コードベースはクリーンでよく整理されており、より高度なエージェントシステムを構築するための優れた基盤として機能します。

## 今後の拡張案

1. **ツールの拡充**: 天気以外の実用的なツールの追加（検索、計算、データベースアクセスなど）
2. **エラーハンドリング**: ツール実行失敗時の適切なエラー処理とリトライ機構
3. **マルチステップ推論**: 複数のツールを組み合わせた複雑なタスクの実行
4. **ツール結果の検証**: ツール実行結果の妥当性チェック
5. **セキュリティ強化**: 環境変数ベースのAPIキー管理
6. **反復的ツール呼び出し**: 複数回のツール実行が必要なタスクへの対応
7. **トリアージエージェント**: ユーザーの意図を理解し、適切なツールを選択する上位エージェント

---

*この分析は Claude Sonnet 4.5 (claude-sonnet-4-5-20250929) によって 2025-11-19 に生成されました*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/quietsleep) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
