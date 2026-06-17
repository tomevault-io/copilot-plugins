## self-review

> 変更完了前の自己レビューチェックリスト（Ralph Wiggum ループ）


# 自己レビュールール

変更を完了する **前に** 以下を確認する。問題があれば修正してから完了すること。

## 必須チェック

1. **レイヤー境界**: 変更ファイルが正しいレイヤーに属し、依存方向を守っているか
   - domain/ → 外部ライブラリ import なし、他レイヤーへの依存なし
   - application/ → infrastructure/ への直接依存なし
   - 迷ったら `npm run check:arch` を実行する

2. **テスト**: 新規ロジックにテストがあるか
   - ドメイン/アプリケーション層の変更にはテストを追加する
   - UI の変更はテスト任意だが、ロジックが含まれる場合は追加する
   - `npm run test` で既存テストが壊れていないことを確認する

3. **型安全性**: TypeScript の型エラーがないか
   - `any` の新規追加は避ける
   - 型アサーション（`as`）の使用は最小限に

4. **ドメインモデルとの整合**: CLAUDE.md のドメインモデル定義と矛盾していないか
   - エンティティの振る舞い（メソッド）がドメイン定義と一致しているか
   - 新しい概念を追加した場合、CLAUDE.md への反映が必要か検討する

## 推奨チェック（PR 前）

5. **コミット粒度**: 1 変更 1 コミットになっているか（git-commit-branch ルール参照）
6. **UI 文言**: 内部用語が画面に露出していないか（ux-copy-and-labels.md 参照）
7. **不要な変更**: デバッグ用 console.log や TODO コメントが残っていないか

---
> Source: [mism-mism/tune](https://github.com/mism-mism/tune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
