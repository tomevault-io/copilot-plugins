## components

> Vue コンポーネントの実装規約


# コンポーネント

- ファイル名・コンポーネント名は PascalCase（既存の `src/components/` に従う）。
- `props` と `emits` は明確に定義し、型注釈を付ける。
- スタイルは可能な限り `scoped` CSS を使用する。
- マークアップは最小限にし、DOM 参照・監視・イベント管理は composable 側へ委譲する。
- 再利用可能な汎用部品は `src/components/common/` に配置する。

---
> Source: [mgfujikura/vue-components](https://github.com/mgfujikura/vue-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
