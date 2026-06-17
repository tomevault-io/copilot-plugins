## xp-design-skill

> Windows XPスタイルのウェブデザインを生成するためのスキル。


# Windows XP デザインスキル (xp_design)

このスキルは、クラシックな Windows XP 風のインターフェースを構築するためのガイドラインとリソースを提供します。

## 設計原則

1. **カラーパレット**: 青色を基調としたグラデーション（XP Blue）を使用せよ。
2. **タイポグラフィ**: `Tahoma` または `Segoe UI` を優先的に使用せよ。
3. **コンポーネント構造**:
    - `.window`: 外枠、境界線、角丸（上部のみ）。
    - `.title-bar`: タイトルバー、グラデーション、白文字、ドロップシャドウ。
    - `.window-content`: コンテンツエリア、パディング。
    - `.xp-button`: 立体的なボタンスタイル。

## 使用方法

XPデザインを適用する際は、以下のファイルをリソースとして読み込むことを推奨する。

- `resources/xp_core.css`: 基本的なCSS変数とクラス。
- `resources/xp_template.html`: 基本構造の雛形。

## 実装上の注意

- ボタンのアクティブ状態（クリック時）には、`box-shadow` を反転させて凹んだ表現を実現せよ。
- 状態表示（ステータス）には、光るドット（LED風）のデザインを使用せよ。

---
> Source: [malken21/xp-design-skill](https://github.com/malken21/xp-design-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
