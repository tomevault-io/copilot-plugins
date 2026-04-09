## tm-js

> D3.js を使用した TM（T字形ER図）データモデル可視化 Web アプリケーション

# TM Data Model Viewer

D3.js を使用した TM（T字形ER図）データモデル可視化 Web アプリケーション

## 開発ルール

- **ドキュメント**: Artifacts（Walkthrough, Implementation Plan 等）は日本語で記述すること。

## 起動方法

```bash
npm install
npm run dev
```

## プロジェクト構成

```
src/
├── types.ts          # Interface定義（JSON入力データ用）
├── models/           # Class定義（描画用）
│   ├── index.ts
│   ├── Name.ts
│   ├── Type.ts
│   ├── Position.ts
│   ├── Size.ts
│   ├── Identifier.ts
│   ├── Attribute.ts
│   ├── EntityIdentifier.ts
│   ├── EntityAttribute.ts
│   ├── Entity.ts
│   ├── RelationshipDetail.ts
│   ├── Relationship.ts
│   └── TMModel.ts
├── renderer.ts       # D3.js描画ロジック
├── exporter.ts       # SVG/PNGエクスポート
├── main.ts           # アプリケーション
└── style.css         # スタイル
data/                 # テストデータ (旧 public/)
vite.config.ts        # Vite設定
```

## 実装済み機能

- D3.js による TM 図描画
- ドラッグ＆ドロップでノード移動
- ズーム・パン機能
- SVG/PNG エクスポート

## データモデル

### EntityType
- `RESOURCE`: リソース
- `RESOURCE-SUBSET`: リソースのサブセット
- `COMPARATIVE`: 対照表
- `CORRESPONDENCE`: 対応表
- `EVENT`: イベント
- `EVENT-SUBSET`: イベントのサブセット
- `RECURSION`: 再帰

### RelationshipType
- `USE`: 使用（参照）
- `SUBSET`: サブセット
- `HDR-DTL`: ヘッダー・明細

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yanqirenshi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yanqirenshi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
