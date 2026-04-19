## debate-arena

> AIディベートアプリ。5つのAIペルソナ（司会・肯定派・否定派・証拠監査・審判）が3フェーズ（立論→検証→収束）の構造化されたディベートを行い、証拠の品質まで監査する議論支援ツール。

# Debate Arena — モバイルアプリ

## プロジェクト概要

AIディベートアプリ。5つのAIペルソナ（司会・肯定派・否定派・証拠監査・審判）が3フェーズ（立論→検証→収束）の構造化されたディベートを行い、証拠の品質まで監査する議論支援ツール。

claude.ai Artifact v8.3からReact Native (Expo) + Supabaseでモバイルネイティブ化。

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | React Native + Expo (Expo Router) + TypeScript |
| スタイリング | NativeWind (Tailwind CSS for RN) |
| 状態管理 | Zustand |
| バックエンド | Supabase (Auth + PostgreSQL + Edge Functions) |
| AI API | Anthropic API (Claude Sonnet 4.6) + Web Search Tool |
| デプロイ | Expo EAS + Supabase Cloud |

## コマンド

```bash
npm start          # Expo開発サーバー起動
npm run ios        # iOS起動
npm run android    # Android起動
npm run typecheck  # 型チェック
npm run lint       # ESLint実行
npm run format     # Prettier実行
```

## ディレクトリ構造

```
src/
├── app/              # Expo Router (ファイルベースルーティング)
├── components/
│   ├── ui/           # 汎用UIコンポーネント
│   ├── debate/       # ディベート固有コンポーネント
│   └── layout/       # レイアウトコンポーネント
├── lib/
│   ├── api/          # APIクライアント (Supabase, Claude)
│   └── debate/       # ディベートエンジン (prompts, engine, parser)
├── hooks/            # カスタムフック
├── stores/           # Zustandストア
└── types/            # 型定義
```

## コンベンション

- 全コメント・ドキュメントは日本語
- コンポーネントはPascalCase、関数はcamelCase
- `src/lib/debate/prompts.ts` のプロンプト定義は改変禁止（v8.3 Artifactからそのまま移植）
- NativeWindでTailwindクラスを使用
- 型定義は `src/types/` に集約
- RLSは最初から有効

## 重要な設計判断

### プロンプト定義（prompts.ts）
v8.3 Artifactから改変なしで移植。以下の3層防壁で出典URL出力を強制:
1. システムプロンプト冒頭に「URLなし事実主張は根拠不足判定」
2. DEBATE_PRINCIPLESに3点セット（URL + 参照箇所要約 + 主張との対応）
3. 全6ラウンド指示に「証拠ルール（厳守）」ブロック

### 議論履歴の2系統化
- `hist`: 最終主張のみ（ディベート中の相手への共有用）
- `fullHist`: 全文（出典URL含む。監査・審判用）

### API呼び出し
- 9回/ディベート（司会1 + 肯定派3 + 否定派3 + 監査1 + 審判1）
- Promise.raceタイムアウト（通常60秒、監査・審判90秒）
- リトライ最大2回

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NekoyaJolly) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
