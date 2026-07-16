## composables

> Composable（src/composable）の実装規約


# Composable

- DOM 参照の取得・監視・クリーンアップを内部で完結させる。
- オプションベースの柔軟な API を提供する。
- UI 側に sentinel 要素や observer 設定を要求しない。
- 命名は `useXxx` とし、責務を 1 つに絞る。

---
> Source: [mgfujikura/vue-components](https://github.com/mgfujikura/vue-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
