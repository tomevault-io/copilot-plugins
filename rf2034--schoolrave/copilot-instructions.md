## schoolrave

> React/Next.js パフォーマンス最適化 - Waterfall、バンドル、再レンダー


# React/Next.js Performance

## CRITICAL: Waterfall排除

- `await` を必要な分岐内に移動
- 独立した操作は `Promise.all()` で並列実行
- API Route: 非依存処理は先に開始、後でawait
- RSC: コンポーネント分割で並列フェッチ（Suspense境界を活用）

## CRITICAL: バンドル最適化

- Barrel importsを避け、直接import（lucide-react, @mui/material等）
- `next/dynamic` で重いコンポーネントを遅延ロード
- 分析/ログは hydration 後に読み込み

## Server-Side

- Server Action: 認証・認可を必ず内部で検証
- `React.cache()` でリクエスト内の重複排除
- クロスリクエストにはLRU cache
- RSC境界: 必要なフィールドのみ渡す（シリアライズ最小化）
- `after()` で非ブロッキング処理（ログ、分析）

## Re-render最適化

- コールバック内でしか使わないstateは購読しない
- `useState` の初期値が重い場合は関数形式
- 派生値はrender中に計算（effectでsetしない）
- `startTransition` で非緊急更新をマーク
- 頻繁に変わる値は `useRef` で保持

## 条件付きレンダリング

- `count && <Badge />` は `count === 0` で "0" が表示される → 三項演算子を使用

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RF2034) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
