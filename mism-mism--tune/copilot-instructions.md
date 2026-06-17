## tdd-implementation

> t_wada式TDD（レッド・グリーン・リファクタ）


# TDD 実装ルール

## レッド・グリーン・リファクタ

実装時は必ずこの順序を守る:

1. **レッド**: 失敗するテストを先に書く
2. **グリーン**: 最小限の実装でテストを通す
3. **リファクタ**: 重複を除き、可読性を高める

## 禁止事項

- テストのない新規ロジックを書かない
- グリーンを飛ばしてリファクタしない（テストが通っている状態でのみリファクタ）

## テストの配置

- ドメイン: `domain/models/**/__tests__/*.test.ts` または同階層の `*.test.ts`
- アプリケーション: `application/usecases/__tests__/*.test.ts`
- インフラ: 必要に応じてモックで分離

## 小さく刻む

1 回の変更では 1 つか 2 つのテスト・最小の実装に留める。大きな塊での実装は避ける。

詳細は `tdd-t-wada` スキルを参照。

---
> Source: [mism-mism/tune](https://github.com/mism-mism/tune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
