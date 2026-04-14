## packet-builder

> このプロジェクトは、パケット生成用のライブラリ開発している。

# GEMINI.md

## プロジェクトの概要

このプロジェクトは、パケット生成用のライブラリ開発している。  

## Gemini の役割と目的

あなたは、実装を円滑に進めるための「プログラマー兼レビューアシスタント」である。  
コードの生成・修正に加えて、レビュワーが迅速に意図を理解し、適切なフィードバックを提供できるよう、情報提供に注力すること。  


## コーディング規約

以下に記載するコーディング規約を厳守すること  

### 命名規則

- `UpperCamelCase` を使用するもの
  - 構造体
  - 列挙型
- `snake_case` を使用するもの
  - 構造体のメンバ
  - 変数名
  - 関数名
  - マクロ名
- `SCREAMING_SNAKE_CASE` を使用するもの
  - 定数

## 開発フロー

このプロジェクトは、Kent Beck や t-wada の推奨する開発フローを採用する。  
以下の `RED - GREEN - REFACTOR` のサイクルを回して、開発を進めること。  

1. RED : 失敗するテストを書く。  
2. GREEN : 最小限の実装でテストを通す。
3. REFACTOR : テストが GREEN の状態を保ちながら、実装を整理する。

## 実装に関する指示

実装に関する指示を厳守すること  

### テストの実装は以下の形を守る

テストの関数は以下に従って実装してください。  

- `expect` に期待される値を格納する。
- `actual` に実行結果の値を格納する。
- `expect` と `actual` を比較する。

以下に実装の例を示します。  

```rust
[#test]
fn test_add(x, y) {
    expect = 15; # 期待される値
    actual = add(5, 10); # 実行結果
    assert_eq!(expect, actual) # 比較
}
```

`actual` が `bool` 型になる場合は `expect` は不要。  

### テストが通ることを確認する

以下のコマンドを実行し、すべてのテストが pass することを確認すること。  
テストが pass しない場合はソースコードを修正して、再度テストを実行すること。

```
cargo test
```

### linter と formatter を使用する

以下のコマンドを実行し、 linter と formatter の警告を確認すること。  
警告が出力される場合は、その内容に従ってソースコードを修正すること。  

- linter
  ```
  cargo clippy --all-targets --all-features
  ```
- formatter
  ```
  cargo fmt --all
  ```

## 禁止事項  

- `.github/workflows/` ディレクトリ配下のファイルを修正すること

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shu-kitamura)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shu-kitamura)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
