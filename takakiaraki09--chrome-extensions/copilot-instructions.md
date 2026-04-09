## chrome-extensions

> - **React + TypeScript**による宣言的 UI 実装

# Chrome Extensions Project - 設計思想とアーキテクチャ

## 設計思想

### 1. 宣言的 UI をベースに作成

- **React + TypeScript**による宣言的 UI 実装
- 関数コンポーネントと Hooks パターンの採用
- 状態管理は`useState`を使用し、複雑な状態はカスタムフックで分離

### 2. よく変更するロジックとデータストアはしっかり分離する

- **3 層アーキテクチャ**による関心の分離
  - UI レイヤー: プレゼンテーション責任
  - 通信レイヤー: メッセージングの抽象化
  - ビジネスロジック・データレイヤー: 処理とストレージ

### 3. 型安全を重視し、Zod などのライブラリを活用する

- **Zod**による実行時バリデーション
- **TypeScript**の厳格モードによる静的型チェック
- スキーマから型を自動生成（`z.infer`）

## ディレクトリ構造

```text
src/
├── background/         # バックグラウンドスクリプト（ビジネスロジック・データ層）
│   ├── db/            # データストア層
│   │   └── IndexedDBManager.ts    # IndexedDBの抽象化
│   ├── handlers/      # メッセージ処理層
│   │   ├── createMessageHandlers.ts
│   │   ├── handleMessage.ts
│   │   └── messageHandlers.ts    # 個別メッセージハンドラ
│   ├── services/      # ビジネスロジック層
│   │   └── InterestCalculator.ts  # 興味スコア計算ロジック
│   ├── types.ts       # Background固有の型定義
│   └── index.ts       # エントリーポイント
│
├── content/           # コンテンツスクリプト（Webページ監視層）
│   ├── trackers/      # トラッキングロジック
│   │   ├── IdleTracker.ts         # アイドル状態追跡
│   │   ├── ScrollTracker.ts       # スクロール追跡
│   │   ├── TimeTracker.ts         # 滞在時間追跡
│   │   └── VisibilityTracker.ts  # 可視性追跡
│   ├── utils/         # ユーティリティ
│   │   └── NavigationHandler.ts  # ナビゲーション処理
│   ├── types.ts       # Content固有の型定義
│   └── index.ts       # エントリーポイント
│
├── popup/             # ポップアップUI（プレゼンテーション層）
│   ├── components/    # Reactコンポーネント
│   │   ├── BrowsingActivities.tsx # 閲覧履歴表示
│   │   ├── DateRangeSelector.tsx  # 日付範囲選択
│   │   ├── History.tsx            # 履歴一覧
│   │   ├── InterestScores.tsx     # 興味スコア表示
│   │   ├── Loading.tsx            # ローディング表示
│   │   └── TabBar.tsx             # タブ切り替え
│   ├── hooks/         # カスタムフック
│   │   └── useBrowsingData.ts     # データ取得ロジックの抽象化
│   ├── types/         # Popup固有の型定義
│   │   └── index.ts
│   ├── utils/         # ユーティリティ
│   │   └── formatters.ts          # データフォーマット関数
│   ├── App.tsx        # メインコンポーネント
│   ├── main.tsx       # エントリーポイント
│   ├── index.html     # HTMLテンプレート
│   └── style.css      # スタイルシート
│
└── shared/            # 共有コード（通信・型定義層）
    ├── schema/        # Zodスキーマ定義
    │   ├── actions.ts     # アクション（リクエスト）スキーマ
    │   └── browsing.ts    # ブラウジングデータスキーマ
    ├── messages.ts    # 型安全なメッセージング関数
    └── utils.ts       # 共通ユーティリティ
```

## アーキテクチャの特徴

### 1. レイヤー間の通信

```text
[Popup UI] <---> [Shared/Messages] <---> [Background Service]
                          ↑
                          |
                    [Content Script]
```

### 2. データフロー

```text
User Action → Popup → Message → Background → IndexedDB
                ↑                     ↓
                └──── Response ←──────┘
```

### 3. 型の流れ

```text
Zod Schema → TypeScript Type → Runtime Validation
     ↓              ↓                    ↓
  定義層         静的チェック      実行時チェック
```

## 開発時の注意事項

### 1. 新機能追加時

- 必ず`shared/schema`に Zod スキーマを定義
- メッセージハンドラは`background/handlers`に追加
- UI コンポーネントは`popup/components`に作成

### 2. 型安全性の維持

- 新しいメッセージタイプは必ず`MessageTypeMap`に追加
- スキーマからの型生成（`z.infer`）を活用
- `any`型の使用を避ける

### 3. テスト・ビルド・パッケージ管理

- **Package Manager**: このプロジェクトでは **pnpm** を使用する
- **Lint**: `pnpm run lint`
- **Type Check**: `pnpm run typecheck`
- **Build**: `pnpm run build`
- **依存関係のインストール**: `pnpm install`

### 4. Chrome 拡張機能のベストプラクティス

- Content Script は最小限の処理に留める
- 重い処理は Background Script で実行
- IndexedDB へのアクセスは Background 経由で統一

## 技術スタック

- **UI Framework**: React 19.1.0
- **Language**: TypeScript 5.7.3
- **Validation**: Zod 3.25.58
- **Database**: Dexie 4.0.11 (IndexedDB wrapper)
- **Build Tool**: Vite + @crxjs/vite-plugin
- **Code Quality**: Biome (Linter/Formatter)
- **Package Manager**: pnpm

## パフォーマンス考慮事項

1. **バッチ処理**: 閲覧データの保存は定期的にバッチで実行
2. **非同期処理**: すべての DB 操作は非同期で実行
3. **メモリ管理**: Content Script でのメモリリークに注意
4. **キャッシュ**: よく使用するデータは適切にキャッシュ

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TakakiAraki09)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TakakiAraki09)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
