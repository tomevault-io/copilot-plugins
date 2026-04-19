## railway-todo-app

> React + Vite + Redux Toolkitを使用したTODOアプリケーション

# Railway Todo App - Cursor Rules

## プロジェクト概要
React + Vite + Redux Toolkitを使用したTODOアプリケーション

## 技術スタック
- **フロントエンド**: React 19.0.0, JSX
- **ビルドツール**: Vite 6.3.2
- **状態管理**: Redux Toolkit 2.7.0, React Redux 9.2.0
- **ルーティング**: React Router DOM 7.3.0
- **フォーム管理**: React Hook Form 7.60.0
- **日付操作**: date-fns 3.0.0
- **HTTP通信**: Axios 1.8.3
- **認証**: React Cookie 8.0.1
- **スタイリング**: CSS, UnoCSS reset
- **フォント**: Inter (@fontsource/inter)

## コーディング規約

### ファイル命名規則
- **コンポーネント**: PascalCase (例: `TaskItem.jsx`, `BackButton.jsx`)
- **ページ**: `index.page.jsx` 形式
- **フック**: `use` プレフィックス + camelCase (例: `useLogin.js`)
- **CSS**: コンポーネント名と同じ (例: `TaskItem.css`)
- **アイコン**: PascalCase + Icon サフィックス (例: `CheckIcon.jsx`)

### ディレクトリ構造
- `src/components/` - 再利用可能なコンポーネント
- `src/pages/` - ページコンポーネント
- `src/hooks/` - カスタムフック
- `src/store/` - Redux store関連
- `src/routes/` - ルーティング設定
- `src/icons/` - アイコンコンポーネント
- `src/utils/` - ユーティリティ関数
- `src/vendor/` - 外部ライブラリ設定

### React コンポーネント規約
1. **関数コンポーネント**を使用
2. **export default**でコンポーネントをエクスポート
3. **React Hooks**を積極的に使用
4. **props destructuring**を行頭で実行
5. **JSX**ファイル拡張子を使用

### Redux Toolkit 規約
1. **createSlice**を使用してstoreを定義
2. **configureStore**でstoreを設定
3. **useDispatch**, **useSelector**でstoreにアクセス
4. **async thunk**でAPI通信を処理

### import規約
1. **React imports**を最初に記述
2. **外部ライブラリ**を次に記述
3. **内部モジュール**を最後に記述
4. **相対パス**には `@/` を使用（Vite設定による）

### CSS規約
1. **コンポーネント単位**でCSSファイルを分離
2. **BEM記法**を推奨
3. **CSS Modules**は使用せず、通常のCSSを使用
4. **レスポンシブデザイン**を考慮

### API通信規約
1. **axios**を使用
2. **vendor/axios.js**で設定を統一
3. **async/await**を使用
4. **エラーハンドリング**を必須実装

### ルーティング規約
1. **React Router DOM v7**を使用
2. **ページベースルーティング**を採用
3. **動的ルーティング**は `[param]` 形式

## 推奨パターン

### コンポーネント作成パターン
```jsx
import React from 'react'
import './ComponentName.css'

function ComponentName({ prop1, prop2 }) {
  // ロジック

  return (
    <div className="component-name">
      {/* JSX */}
    </div>
  )
}

export default ComponentName
```

### カスタムフック作成パターン
```javascript
import { useState, useEffect } from 'react'

export function useCustomHook() {
  const [state, setState] = useState(null)

  useEffect(() => {
    // effect logic
  }, [])

  return { state, setState }
}
```

### Redux Slice作成パターン
```javascript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'

export const asyncAction = createAsyncThunk(
  'slice/asyncAction',
  async (payload) => {
    // async logic
  }
)

export const sliceSlice = createSlice({
  name: 'slice',
  initialState: {},
  reducers: {},
  extraReducers: (builder) => {
    builder.addCase(asyncAction.fulfilled, (state, action) => {
      // handle success
    })
  }
})
```

## 禁止事項
- **class component**の使用
- **React.createElement**の直接使用
- **直接的なDOM操作**
- **console.log**の本番環境への残存
- **魔法の数値**の使用（定数化を推奨）

## パフォーマンス考慮事項
- **React.memo**でのコンポーネント最適化
- **useCallback**, **useMemo**の適切な使用
- **lazy loading**の実装検討
- **bundle size**の最適化

## テスト方針
- **React Testing Library**を使用
- **ユーザー中心**のテストを作成
- **integration test**を優先
- **mocking**は最小限に

## セキュリティ
- **XSS**対策の実装
- **認証トークン**の適切な管理
- **API endpoints**の保護
- **環境変数**での機密情報管理

## Git 規約
- **コミットメッセージ**は日本語で明確に
- **feature branch**での開発
- **Pull Request**でのコードレビュー
- **conventional commits**の採用検討 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wangchangdog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
