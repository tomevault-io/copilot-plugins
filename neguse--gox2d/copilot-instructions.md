## gox2d

> Box2D v3.1.1 の C ソースコードを Go にピュア移植するプロジェクト。

# gox2d AGENTS

Box2D v3.1.1 の C ソースコードを Go にピュア移植するプロジェクト。

## ドキュメント

- `PORTING_GUIDE.md` — 逐語移植手順書（翻訳パターン辞書・ファイル対応表・全22ステップ）
- `docs/tasks.md` — 作業キュー
- `docs/done.md` — 完了記録

## 参照ソース

- `refs/box2d/` — Box2D v3.1.1 (submodule, タグ固定)
- `refs/box2d/src/` — 移植元 C ソース（約29,500行）
- `refs/box2d/include/box2d/` — 移植元公開ヘッダ（約4,600行）
- `refs/box2d/test/` — 移植元テスト

## コマンド

```sh
go build ./box2d/                    # ビルド
go test ./box2d/                     # テスト
go vet ./box2d/                      # 静的解析
bash scripts/run-tests.sh            # テスト（スクリプト経由）
```

## 実装方針

- C の関数・構造体を **1:1 逐語翻訳** する（PORTING_GUIDE.md の翻訳パターン辞書に従う）
- スカラーパス（`BOX2D_DISABLE_SIMD` 相当）のみ移植する。SIMDコードは翻訳しない
- 振る舞い変更には必ずテストを追加/更新する
- 対応する C テスト（`vendor/box2d/test/`）を Go に移植してパスさせる
- Go 慣用化・最適化はフェーズ1完了後に行う。まず忠実な翻訳を優先する
- 互換性は考慮しない。API は Box2D v3 と 1:1 対応させる
- Tracy プロファイラ、プラットフォーム分岐は移植しない

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neguse)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/neguse)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
