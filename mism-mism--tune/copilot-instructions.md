## frontend-screen-composition

> TUNEのフロントエンド画面構成ルール（ブランド準拠）


# Frontend Screen Composition (TUNE)

`docs/brand-guidelines.md` を唯一の基準として、画面は「静かで育つ知性」を伝える構成にする。

## 1) 情報設計
- 1画面で主目的は1つに絞る（依頼・評価・蓄積のどれか）。
- ページ先頭に「この画面で何が育つか」を1文で示す。
- 主操作（送信/保存/次へ）は1視線で見つかる位置に置く。

## 2) 構造
- `AppNav` の直下は `main id="main-content"` を使う。
- セクションは意味単位で `section` を使い、カードは補助表現として使う。
- 空状態には必ず次の一歩（リンク or ボタン）を置く。

## 3) 文言ルール
- ユーザー向け語彙は「引き出し」「育つ」「効く」を優先する。
- Persona/Policy などの内部用語はUI文言に出さない。
- 成功メッセージは「完了」だけで終わらせず、次に起こることを示す。

## 4) 視覚ルール
- 色・余白・タイポは `globals.css` のトークンを使い、直書き乱立を避ける。
- 派手さより可読性を優先し、装飾は文脈説明に寄与する場合のみ追加する。
- 主要状態（通常/hover/focus/error/success）を必ず区別可能にする。

## 5) 実装時チェック
- 新規/改修画面で次を満たすこと:
  - 依頼→評価→蓄積のどこに位置する画面か説明できる
  - 主操作が明確で、操作後の遷移が予測できる
  - 専門用語なしで非エンジニアが理解できる

## Example
```tsx
<div className="page-container">
  <AppNav />
  <main id="main-content" className="content-area">
    <section className="card">{/* 画面の主目的 */}</section>
    <section className="card">{/* 次アクション */}</section>
  </main>
</div>
```

---
> Source: [mism-mism/tune](https://github.com/mism-mism/tune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
