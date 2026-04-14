## test-front-mock

> プライマリカラー: blue-600 (ホバー時: blue-700)

# ランディングページ作成で使用したTailwind CSSテクニック

## デザインシステムと統一ルール

### 1. カラーパレット
```
プライマリカラー: blue-600 (ホバー時: blue-700)
セカンダリカラー: gray-800 (ホバー時: gray-900)
背景色: gray-50 (ページ全体)
テキストカラー:
  - 見出し: text-gray-900
  - 本文: text-gray-600
  - リンク: text-gray-300 (フッター内)
```

### 2. スペーシング規則
```
セクションパディング:
  - メインセクション: py-16
  - サブセクション: py-12
  - 小セクション: py-8
  
コンテナパディング:
  - px-4 sm:px-6 lg:px-8 (レスポンシブ対応)
```

### 3. タイポグラフィ
```
ページタイトル: text-4xl md:text-5xl font-bold
セクション見出し: text-3xl font-bold
サブ見出し: text-2xl font-bold
説明文: text-xl (重要) / text-lg (通常)
```

### 4. ボタンスタイル
```
プライマリボタン:
  bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-8 rounded-lg transition

セカンダリボタン:
  bg-white hover:bg-gray-50 text-gray-800 font-bold py-3 px-8 rounded-lg border border-gray-300 transition

ダークボタン:
  bg-gray-800 hover:bg-gray-900 text-white font-bold py-3 px-8 rounded-lg transition

ナビゲーションCTA:
  bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-lg transition

重要なCTA:
  上記 + transform hover:scale-105
```

### 5. レイアウト構造
```html
<!-- ページ基本構造 -->
<body class="min-h-screen flex flex-col bg-gray-50">
  <nav>...</nav>
  <main class="flex-grow">...</main>
  <footer>...</footer>
</body>

<!-- セクション構造 -->
<section class="py-16">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <!-- コンテンツ -->
  </div>
</section>
```

### 6. コンポーネントパターン

#### ナビゲーション
```html
<nav class="bg-white shadow-lg sticky top-0 z-50">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex justify-between h-16">
      <!-- ロゴ、メニュー、CTAボタン -->
    </div>
  </div>
</nav>
```

#### ヒーローセクション
```html
<section class="bg-gradient-to-r from-blue-600 to-indigo-700 text-white py-16">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="text-center">
      <h1 class="text-4xl md:text-5xl font-bold mb-4">タイトル</h1>
      <p class="text-xl opacity-90 max-w-3xl mx-auto">説明文</p>
    </div>
  </div>
</section>
```

#### カード
```html
<div class="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-xl transition-shadow">
  <!-- カード内容 -->
</div>
```

### 7. アニメーション規則
```
通常のトランジション: transition
ボタンホバー: transition transform hover:scale-105
カードホバー: transition-shadow
```

### 8. レスポンシブ規則
```
モバイルファースト
ブレークポイント:
  - sm: 640px以上
  - md: 768px以上  
  - lg: 1024px以上
  - xl: 1280px以上

表示/非表示:
  - hidden md:flex (モバイルで非表示、タブレット以上で表示)
  - block md:hidden (モバイルで表示、タブレット以上で非表示)
```

### 9. アクセシビリティ
```
- 必須項目: <span class="text-red-500">*</span>
- フォーカス状態: focus:ring-2 focus:ring-blue-500 focus:border-transparent
- ホバー状態: 必ずtransitionを追加
```

### 10. 影の使用
```
ナビゲーション: shadow-lg
カード（通常）: shadow-md
カード（ホバー）: shadow-xl
重要なカード: shadow-xl (デフォルト) hover:shadow-2xl
```

---

## レイアウト構造

### Flexboxを使った全体構成
```html
<body class="min-h-screen flex flex-col">
```
- `min-h-screen`: ビューポート全体の高さを確保
- `flex flex-col`: 縦方向のFlexコンテナ
- ヒーローセクションに`flex-grow`を適用してフッターを下部に固定

### スティッキーナビゲーション
```html
<nav class="bg-white shadow-lg sticky top-0 z-50">
```
- `sticky top-0`: スクロール時に上部に固定
- `z-50`: 他の要素より前面に表示
- `shadow-lg`: 影で浮き上がり効果

## レスポンシブデザイン

### ブレークポイントの活用
- `hidden md:flex`: モバイルで非表示、タブレット以上で表示
- `sm:flex-row`: モバイルで縦並び、小画面以上で横並び
- `text-4xl md:text-6xl`: 画面サイズに応じてフォントサイズ変更

### コンテナの最大幅制限
```html
<div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
```
- `max-w-7xl`: 最大幅1280px
- `mx-auto`: 中央揃え
- 画面サイズに応じたパディング調整

## 装飾効果

### グラデーション背景
```html
<section class="bg-gradient-to-br from-blue-50 to-indigo-100">
```
- `bg-gradient-to-br`: 左上から右下へのグラデーション
- `from-blue-50 to-indigo-100`: 薄い青から薄い藍色へ

### ホバーエフェクト
```html
<button class="hover:bg-blue-700 transition transform hover:scale-105">
```
- `transition`: スムーズなアニメーション
- `hover:scale-105`: ホバー時に5%拡大
- `hover:bg-blue-700`: ホバー時の色変更

## グリッドレイアウト

### フッターの4カラム構成
```html
<div class="grid grid-cols-1 md:grid-cols-4 gap-8">
```
- モバイル: 1カラム
- タブレット以上: 4カラム
- `gap-8`: 要素間の間隔

## スペーシング

### 一貫性のあるスペース
- `space-x-8`: 横方向の要素間隔
- `space-y-2`: 縦方向の要素間隔
- `py-20`: 上下の大きなパディング
- `px-4`: 左右の基本パディング

## ボタンデザイン

### プライマリボタン
```html
<button class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-8 rounded-lg">
```

### セカンダリボタン
```html
<button class="bg-white hover:bg-gray-50 text-gray-800 font-bold py-3 px-8 rounded-lg border border-gray-300">
```

## テキストスタイル

### 階層的なタイポグラフィ
- 見出し: `text-4xl md:text-6xl font-bold`
- 説明文: `text-xl text-gray-600`
- リンク: `text-gray-300 hover:text-white`

## ユーティリティクラスの組み合わせ

### センタリングパターン
- `flex items-center justify-center`: Flexでの中央揃え
- `text-center`: テキストの中央揃え
- `mx-auto`: ブロック要素の中央揃え

### 可視性の制御
- `hidden md:flex`: レスポンシブな表示/非表示
- モバイルファーストアプローチ

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nakatatsu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nakatatsu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
