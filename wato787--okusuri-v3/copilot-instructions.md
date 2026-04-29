## okusuri-v3

> お薬管理アプリのフロントエンド開発ガイドラインに従って開発してください。

# フロントエンド開発

お薬管理アプリのフロントエンド開発ガイドラインに従って開発してください。

## 🎨 技術スタック

### コア技術
- **フレームワーク**: React 19
- **ビルドツール**: Vite
- **言語**: TypeScript
- **スタイリング**: Tailwind CSS
- **UI コンポーネント**: Radix UI

### 追加ライブラリ
- **アニメーション**: Framer Motion
- **アイコン**: Lucide React
- **通知**: React Hot Toast
- **ユーティリティ**: clsx, tailwind-merge
- **日付処理**: date-fns

## 📁 ディレクトリ構造

```
frontend/
├── src/
│   ├── App.tsx              # アプリケーションエントリーポイント
│   ├── main.tsx             # アプリケーション初期化
│   ├── components/          # 再利用可能なコンポーネント
│   │   ├── layout/          # レイアウトコンポーネント
│   │   ├── navigation/      # ナビゲーションコンポーネント
│   │   └── ui/              # UI コンポーネント
│   ├── views/               # ページコンポーネント
│   ├── lib/                 # ライブラリ・ユーティリティ
│   ├── styles/              # スタイルファイル
│   └── utils/               # アプリケーション固有のユーティリティ
```

## 🚀 アプリケーション設定

### エントリーポイント (main.tsx)
```typescript
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

### アプリケーションコンポーネント (App.tsx)
```typescript
import { Home } from "@/views/home";
import "@/styles/globals.css";

export function App() {
  return <Home />;
}

export default App;
```

## 🎨 スタイリング

### Tailwind CSS設定
- ユーティリティファーストのアプローチ
- カスタムカラーパレットの定義
- レスポンシブデザインの実装

### グローバルスタイル
- ベーススタイルの設定
- フォントファミリーの統一
- コンポーネントクラスの定義

## 🧩 コンポーネント設計

### UI コンポーネント
- **Button**: バリアント（default, destructive, outline, secondary, ghost, link）
- **Card**: CardHeader, CardTitle, CardContent
- **Input**: フォーム入力コンポーネント
- **forwardRef**: refの適切な処理

### レイアウトコンポーネント
- **Header**: アプリケーションヘッダー
- **BottomNavigation**: モバイル用ナビゲーション
- **HamburgerMenu**: ハンバーガーメニュー

### コンポーネント設計原則
- 単一責任の原則
- Propsの型安全性
- デフォルト値の適切な設定
- アクセシビリティの考慮

## 📱 ページコンポーネント

### Home ページ
- 今日のお薬表示
- 服用状況の管理
- 新しい薬の追加機能

### ページ設計原則
- モバイルファースト
- レスポンシブデザイン
- ユーザビリティの向上

## 🔧 ユーティリティ

### ユーティリティ関数
- **cn**: クラス名の結合
- **formatDate**: 日付のフォーマット
- **formatTime**: 時刻のフォーマット

### API クライアント
- **ApiClient**: HTTPリクエストの管理
- エラーハンドリング
- レスポンスの型安全性

## 🧪 テスト

### テストの実行
```bash
# 全テスト実行
bun test

# カバレッジ付きテスト
bun test --coverage

# 特定のテストファイル実行
bun test utils.test.ts
```

### テスト例
- ユーティリティ関数のテスト
- コンポーネントのテスト
- APIクライアントのテスト

## 🚀 ビルド・デプロイ

### ビルド設定
- Vite設定の最適化
- ソースマップの生成
- バンドルサイズの最適化

### 利用可能なスクリプト
```bash
# 開発サーバー起動
bun run dev

# ビルド
bun run build

# プレビュー
bun run preview

# 型チェック
bun run typecheck

# リント実行
bun run lint

# テスト実行
bun test
```

## 📱 レスポンシブデザイン

### ブレークポイント
- **sm**: 640px（スマートフォン）
- **md**: 768px（タブレット）
- **lg**: 1024px（デスクトップ）
- **xl**: 1280px（大画面デスクトップ）

### レスポンシブコンポーネント例
- グリッドレイアウトの調整
- テキストサイズの変更
- ナビゲーションの切り替え

## 🎨 アニメーション

### Framer Motion の使用
- ページ遷移のアニメーション
- コンポーネントの表示アニメーション
- インタラクションのフィードバック

### アニメーション原則
- パフォーマンスを考慮
- アクセシビリティに配慮
- 適度な使用

## 🔧 開発ガイドライン

### コンポーネント作成
1. 型定義を最初に作成
2. Propsのインターフェースを定義
3. デフォルト値を設定
4. テストを記述

### スタイリング
1. Tailwind CSSを優先使用
2. カスタムCSSは最小限に
3. レスポンシブデザインを考慮
4. アクセシビリティを重視

### 状態管理
1. useState/useEffectを基本とする
2. 複雑な状態はContext APIを検討
3. カスタムフックでロジックを分離
4. 状態の更新は関数型更新を使用

## 🔗 関連ドキュメント

詳細なフロントエンド開発については `docs/frontend.md` を参照してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wato787) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
