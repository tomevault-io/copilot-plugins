## oogiri-odai-factory

> OpenAI GPT、Claude、Geminiの3つのAIを活用して、多様で創造的な大喜利のお題を大量生成するNext.jsアプリケーション

# 大喜利お題生成アプリ - Claude Code ガイド

## プロジェクト概要
OpenAI GPT、Claude、Geminiの3つのAIを活用して、多様で創造的な大喜利のお題を大量生成するNext.jsアプリケーション

## 設計ドキュメント参照
詳細な設計については以下のドキュメントを参照してください：
- **[メイン設計](./docs/README.md)** - アプリ概要、技術スタック、システム構成
- **[API設計](./docs/api-design.md)** - エンドポイント、データ型、プロンプト戦略
- **[UI設計](./docs/ui-design.md)** - レイアウト、コンポーネント、デザインシステム

## 開発環境セットアップ

### 必要なAPI キー
以下のAPIキーを `.env.local` に設定：
```bash
OPENAI_API_KEY=sk-proj-XXX
ANTHROPIC_API_KEY=sk-ant-XXX
GEMINI_API_KEY=XXX
```

### 依存関係
すでにインストール済み：
- `openai` - OpenAI API クライアント
- `@anthropic-ai/sdk` - Claude API クライアント
- `@google/generative-ai` - Gemini API クライアント

### 開発コマンド
```bash
pnpm run dev          # 開発サーバー起動
pnpm run build        # プロダクションビルド
pnpm run check        # Biome によるコードチェック
```

## 実装フェーズ

### Phase 1: 基盤構築
1. **型定義作成** (`src/types/index.ts`)
   - API リクエスト/レスポンス型
   - お題データ型
   - UI状態管理型

2. **AIクライアント実装** (`src/lib/`)
   - `openai.ts` - OpenAI API クライアント
   - `claude.ts` - Claude API クライアント  
   - `gemini.ts` - Gemini API クライアント
   - `prompts.ts` - プロンプト定義

3. **環境変数設定**
   - `.env.local` ファイル作成
   - API キー設定確認

### Phase 2: API層実装
4. **API Routes作成** (`src/app/api/`)
   - `openai/route.ts` - OpenAI エンドポイント
   - `claude/route.ts` - Claude エンドポイント
   - `gemini/route.ts` - Gemini エンドポイント
   - `generate-all/route.ts` - 一括生成エンドポイント

5. **エラーハンドリング**
   - 統一的なエラーレスポンス
   - レート制限対応
   - バリデーション実装

### Phase 3: UI実装
6. **基本レイアウト更新**
   - `src/app/layout.tsx` - メタデータ、言語設定
   - `src/app/globals.css` - Tailwind設定、カスタムスタイル

7. **コンポーネント作成** (`src/components/`)
   - `OdaiGenerator.tsx` - メイン生成コンポーネント
   - `ControlPanel.tsx` - 設定パネル
   - `AISelector.tsx` - AI選択
   - `CategorySelector.tsx` - カテゴリ選択
   - `OdaiCard.tsx` - お題表示カード
   - `ResultsArea.tsx` - 結果表示エリア

8. **メインページ実装**
   - `src/app/page.tsx` - メインページ構成
   - 状態管理（useState, useEffect）
   - API呼び出し処理

### Phase 4: 機能統合・UX改善
9. **高度な機能実装**
   - お気に入り機能（localStorage）
   - コピー機能（Clipboard API）
   - 再生成機能
   - ローディング状態管理

10. **レスポンシブ対応**
    - モバイル最適化
    - タブレット対応
    - デスクトップレイアウト

11. **エラー処理・UX向上**
    - トースト通知
    - エラー状態表示
    - 空状態表示
    - アニメーション追加

### Phase 5: 最適化・テスト
12. **パフォーマンス最適化**
    - コンポーネント最適化
    - API呼び出し最適化
    - バンドルサイズ最適化

13. **テスト・デバッグ**
    - 各AI API動作確認
    - UI/UX テスト
    - エラーケース確認

## ファイル構成（実装後）
```
src/
├── app/
│   ├── api/
│   │   ├── openai/route.ts
│   │   ├── claude/route.ts
│   │   ├── gemini/route.ts
│   │   └── generate-all/route.ts
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── OdaiGenerator.tsx
│   ├── ControlPanel.tsx
│   ├── AISelector.tsx
│   ├── CategorySelector.tsx
│   ├── OdaiCard.tsx
│   └── ResultsArea.tsx
├── lib/
│   ├── openai.ts
│   ├── claude.ts
│   ├── gemini.ts
│   └── prompts.ts
└── types/
    └── index.ts
```

## 重要な実装ポイント

### APIキー管理
- 環境変数での管理徹底
- クライアントサイドに露出しない
- エラーハンドリングでAPIキー不足を適切に処理

### プロンプト設計
- 各AIの特性を活かした差別化
- カテゴリ・難易度に応じた動的プロンプト
- 一貫性のある出力フォーマット

### UI/UX
- レスポンシブデザイン必須
- ローディング状態の適切な表示
- エラー状態の分かりやすい表示
- アクセシビリティ対応

### エラーハンドリング
- ネットワークエラー
- API制限エラー
- パースエラー
- 認証エラー

## 次のステップ
このガイドに従って実装を進めてください。各フェーズを順番に実装し、動作確認を行いながら進めることを推奨します。

質問や課題が発生した場合は、設計ドキュメントを参照するか、具体的な実装について相談してください。

## 開発時の注意

- 修正後は必ず `pnpm run check` を実行してコードの整合性を確認してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tocomi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
