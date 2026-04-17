## ai-design-system

> 本ドキュメントは Dropdown コンポーネントの設計・API・運用ルールです。AI運用原則とトークン規約（ai-operations.mdc）に従い、全スタイルはトークンのみで指定します（px/rgba直書き禁止）。

# Dropdown（メニュー）

本ドキュメントは Dropdown コンポーネントの設計・API・運用ルールです。AI運用原則とトークン規約（ai-operations.mdc）に従い、全スタイルはトークンのみで指定します（px/rgba直書き禁止）。

## 目的
- クリックで開閉するメニュー UI を提供する
- ボタン基盤（button-base）と寸法を揃え、既存のボタンサイズ体系に準拠
- 階層（サブメニュー）や区切り、ショートカット表示などを一貫した見た目で提供

## 依存・ベース
- トリガー: `ds-button-base ds-btn ds-btn--outline ds-btn--neutral ds-btn--{sm|md|lg}` を適用
- スタイル読み込み: `components/dropdown/Dropdown.css`（内部で `../button/Button.css` を import）

## API
```
Dropdown({
  label: string,                 // トリガーのラベル
  leftIcon?: string,             // 左側アイコン（HTML/SVG）
  rightIcon?: string,            // 右側アイコン（HTML/SVG）
  size?: 'sm' | 'md' | 'lg',     // 既定: 'md'（Buttonと同寸法）
  defaultOpen?: boolean,         // 初期オープン（Story/デバッグ用途）
  items: Array<
    | { type?: 'item', label: string, href?: string, iconLeft?: string, meta?: string, disabled?: boolean, danger?: boolean, submenu?: Array<{ label: string, href?: string }> }
    | { type: 'section', label: string }
    | { type: 'separator' }
  >
})
```

### ふるまい
- クリックで開閉（`.is-open` 切替）
- メニュー外クリック・Esc でクローズ（ドキュメント全体に一度だけリスナーを付与）
- サブメニューは親項目の hover/focus-within で右側へフライアウト
  - `margin-left: var(--spacing-padding-4)` のバッファと、ヒットエリアを疑似要素で確保

### アクセシビリティ
- トリガー: `<button type="button">`（`aria-label` は必要に応じて）
- メニュー: `role="menu"`、項目: `role="menuitem"`
- キーボード: Esc で閉じる（将来的に矢印ナビゲーションの拡張を検討）

## デザイン仕様（トークン）
- パネル（メニュー本体）
  - 位置: トリガー直下（`top: 100%; left: 0`）
  - 余白: 上下 `var(--spacing-padding-12)`、左右 0
  - 背景: `--color-semantic-neutral-50-*`
  - 枠線: `--color-semantic-divider-middle-*`
  - 角丸: `--radius-md`
  - 影: `--shadow-box-shadow-lg-*`
- 行（項目）
  - 余白: 上下 `var(--spacing-padding-8)`、左右 `var(--spacing-padding-24)`
  - フォント: `--font-size-16 / --font-line-height-24`
  - hover 背景: `--color-semantic-neutral-100-*`
  - `danger`: テキスト色 `--color-semantic-negative-600-*`
  - `disabled`: `opacity: --opacity-50`
- セクション
  - 余白: 上下 `var(--spacing-padding-8)`、左右 `var(--spacing-padding-24)`
  - 文字色: `--color-semantic-text-low-*`
- セパレーター
  - `border-top: 1px solid --color-semantic-divider-middle-*`
- ショートカット(meta)
  - 文字色: `--color-semantic-text-low-*`
  - フォント: `--font-size-14`
  - 左余白: `var(--spacing-padding-24)`
- サブメニュー
  - 親項目右側へフライアウト（左から右）、左側バッファ確保
  - パネル余白・背景・角丸・影は親と同ルール

## 実装ファイル
- `components/dropdown/Dropdown.js`
- `components/dropdown/Dropdown.css`
- `storybook/stories/Dropdown.stories.js`
  - `Overview`: 基本例
  - `CompleteMatrix`: `State × Size`（sm/md/lg と default/open）

## 使用例
```js
Dropdown({
  label: 'メニュー',
  size: 'md',
  rightIcon: '<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M7 10l5 5 5-5z"/></svg>',
  items: [
    { type: 'section', label: 'チーム' },
    { label: 'ユーザー招待', submenu: [ { label: 'メール' }, { label: 'メッセージ' }, { label: 'その他...' } ] },
    { type: 'separator' },
    { label: '設定', href: '#' },
    { label: 'ログアウト', danger: true }
  ]
})
```

## アンチパターン
- hover だけで開閉させる（クリック開閉が既定）
- パネル左右に独自余白を追加する（左右 0、行側で左右 24 に統一）
- px/rgba の直書き、トークン未使用

## QA
- 「ボタンとサイズが揃うか」→ `size` は sm/md/lg に対応し、Button と同寸法
- 「外側クリックで閉じるか」→ ドキュメントに 1 度だけグローバルリスナーを登録
- 「テーブル内で見切れないか」→ Matrix では `_matrix.css` の `overflow: visible` を併用

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/livyn-inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
