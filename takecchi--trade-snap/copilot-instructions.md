## trade-snap

> - カスタム hooks は、それを必要とするコンポーネント内で直接呼ぶ

# コンポーネント設計ルール

### hooks はコンポーネントに閉じ込める
- カスタム hooks は、それを必要とするコンポーネント内で直接呼ぶ
- 親コンポーネントに hooks をまとめて props で渡すパターンは避ける
- 各コンポーネントが自分の状態管理に責任を持つ

### 例: TradeSnapEditor
```
TradeSnapEditor               ← 全体のオーケストレーション（useImageUpload, useCanvasLabels, useCanvasExport）
  EditorLayout                ← レイアウト構造のみ（header + main + toolbar）
    AppHeader                 ← ヘッダー表示（props のみ）
    CanvasStage               ← Konva キャンバス（props のみ）
    ImageUploader             ← アップロード UI（props のみ）
    LabelToolbar              ← ラベルボタン + ズーム（props のみ）
```

**NG パターン:**
```
TradeSnapEditor ← 全ての hooks を集めてバケツリレーで渡す
  CanvasWrapper ← 中間コンポーネントが props を中継するだけ
    CanvasStage ← 最終的に props を受け取る
```

### コンポーネント分割の基準
- 責務が異なるなら別コンポーネントにする
- レイアウトとロジックは分離する（EditorLayout はロジックを持たない）
- UI コンポーネント（packages/ui）は外部 API に依存しない。Story では mock データを直接定義する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/takecchi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
