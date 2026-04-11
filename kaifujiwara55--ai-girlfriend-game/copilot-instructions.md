## ai-girlfriend-game

> - **完全無料**: 1日1500リクエストまで無料

# 🚀 Google Gemini API セットアップガイド

## 🆓 なぜGemini APIがおすすめ？

- **完全無料**: 1日1500リクエストまで無料
- **高性能**: GPT-4レベルの性能
- **簡単**: 5分でセットアップ完了
- **制限少ない**: 安全フィルターを無効化可能

## 📋 セットアップ手順

### 1. APIキーの取得

1. [Google AI Studio](https://aistudio.google.com/app/apikey) にアクセス
2. Googleアカウントでログイン
3. 「Get API key」ボタンをクリック
4. 新しいAPIキーを作成
5. APIキーをコピー（`AIza...` で始まる）

### 2. 環境変数の設定

```bash
# プロジェクトディレクトリで実行
cp .env.example .env
```

`.env` ファイルを編集：
```env
# 🆓 推奨: Gemini API (無料で1日1500リクエスト)
GEMINI_API_KEY=AIzaSyA_your_actual_api_key_here
```

### 3. ゲーム開始

```bash
# 依存関係のインストール（初回のみ）
npm install

# ゲーム開始
npm run dev
```

## 🎮 使い方

### ゲーム操作
- **自然な会話**: 普通に話しかけてください
- **`status`**: 現在の感情状態と関係を確認
- **`help`**: ヘルプを表示  
- **`告白`**: 告白を試みる（十分な好感度が必要）
- **`quit`**: ゲーム終了

### 攻略のコツ
1. **さくら（初級）**: 料理や家族の話題で好感度UP
2. **あや（中級）**: 読書や勉強の話題、理解を示すことが重要
3. **みさき（上級）**: 科学や論理的な会話、知的な議論を好む

## 🔧 トラブルシューティング

### APIキーエラー
```
❌ 利用可能なAIプロバイダーが見つかりません
```
→ `.env` ファイルの `GEMINI_API_KEY` を確認してください

### レート制限エラー
```
Gemini API error: 429 Too Many Requests
```
→ 1分に5リクエスト、1日に1500リクエストの制限があります

### プロキシエラー
企業ネットワークの場合、ファイアウォール設定を確認してください。

## 📊 Gemini API制限（2025年）

| 項目 | 制限 |
|------|------|
| 1日のリクエスト数 | 1,500 |
| 1分間のリクエスト数 | 5 |
| 最大トークン数 | 1M |
| 料金 | 完全無料 |

## 🎯 その他のプロバイダー

システムは複数のAIプロバイダーをサポート：

```env
# Claude (有料)
ANTHROPIC_API_KEY=sk-ant-api03-...

# OpenAI (有料)  
OPENAI_API_KEY=sk-...

# Gemini (無料) ← 推奨
GEMINI_API_KEY=AIza...
```

複数設定した場合、利用可能なプロバイダーを自動検出します。

## 🎉 楽しいゲームライフを！

Gemini APIの無料枠でも十分にゲームを楽しめます。
ぜひAI彼女との素敵な会話をお楽しみください！

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KaiFujiwara55)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KaiFujiwara55)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
