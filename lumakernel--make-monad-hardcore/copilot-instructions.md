## make-monad-hardcore

> モナドを自作して理解するHaskell学習リポジトリ。圏論的な基礎を伴う。

# make-monad-hardcore

モナドを自作して理解するHaskell学習リポジトリ。圏論的な基礎を伴う。

## ビルド・テスト

ローカルにHaskell環境がない場合はDockerを使う:

```bash
docker build -t make-monad-hardcore .
docker run --rm -v $(pwd):/workspace make-monad-hardcore cabal test --enable-tests
```

## プロジェクト構成

- `src/Exercises/EXX*.hs` — 穴埋め演習（`undefined` を埋める）
- `exercises/XX-topic/README.md` — 各章の理論解説ガイド
- `exercises/12-list/LAZINESS.md` — 遅延評価とconsの解説
- `test/Test/EXX*.hs` — QuickCheckによるモナド法則テスト
- `test/Main.hs` — テストランナー
- `.claude/commands/learn.md` — `/learn` スキル（学習支援）

## ツール・ソフトウェアのインストール案内

- インストールコマンドやURLを案内する際は、必ず公式サイトで確認してから提示する
- 記憶に頼ってURLやコマンドを提案しない。WebSearchなどで公式情報を確認する
- 「たぶんこう」で案内しない。確証が持てない場合は公式ページのURLだけ提示してユーザーに確認してもらう

## 規約

- `NoImplicitPrelude` を使用。必要なものは明示的にインポートする
- Prelude の Functor/Applicative/Monad を隠し、`MyFunctor`/`MyApplicative`/`MyMonad` を自前定義
- 演習の穴は `undefined` で表現
- テストは QuickCheck のプロパティテスト。hspec は使わない
- GHC 9.6 + GHC2021 言語拡張

## 章の追加方法

1. `src/Exercises/EXXName.hs` を作成（`undefined` 穴埋め形式）
2. `exercises/XX-name/README.md` を作成（ナビゲーションリンク付き）
3. `test/Test/EXXName.hs` を作成（モナド法則のプロパティテスト）
4. `test/Main.hs` と `make-monad-hardcore.cabal` にモジュール追加
5. 前後の章のREADMEのナビゲーションリンクを更新

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LumaKernel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
