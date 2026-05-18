## b-system

> - コメントはすべて日本語で記述してください。

# プロジェクトのコーディングルール

## 言語
- コメントはすべて日本語で記述してください。

## スタイル設計
- スタイルはインライン (`style={{ ... }}`) で書かず、必ず外部のCSSファイルに記述してください。
- CSSのクラス名はすべて **camelCase (キャメルケース)** で統一してください。
- 要素の状態を示すクラス名には、接頭辞 `is` や `selected` を活用してください（例: `isActive`, `isSelected`, `isCompleted`）。

---
> Source: [KodaTemari/b-system](https://github.com/KodaTemari/b-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
