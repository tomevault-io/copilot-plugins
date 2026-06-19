## astro-import-tree

> Astro のページ(例: src/pages/index.astro)の URL パスと、インポートツリーを AST を使い解析して、構造データやテキストを生成するライブラリです。

# このプロジェクトは

Astro のページ(例: src/pages/index.astro)の URL パスと、インポートツリーを AST を使い解析して、構造データやテキストを生成するライブラリです。

## 利用技術

- TypeScript

## 例

```typescript
import AstroImportTree from 'astro-import-tree';

const tree = await AstroImportTree.parse({
  dir: './',
})

console.log(tree)

console.log(AstroImportTree.toLLM(tree));

--- 結果 (オブジェクト)

{
  "pages": [
    {
      "path": "/",
      "imports": [
        "@src/pages/index.astro",
        "@src/components/Header.astro",
        "@src/components/Hero.astro",
        "@src/components/Hero/Image.astro",
        "@src/components/Footer.astro"
      ]
    },
    {
      "path": "/about",
      "imports": [
        "@src/pages/about.astro",
        "@src/components/Header.astro",
        "@src/components/Footer.astro"
      ]
    }
  ]
}

--- 結果 (LLM用テキスト)

## /, /index.html

- @src/pages/index.astro
- @src/components/Header.astro
- @src/components/Hero.astro
- @src/components/Hero/Image.astro
- @src/components/Footer.astro

## /about, /about/index.html

- @src/pages/about.astro
- @src/components/Header.astro
- @src/components/Footer.astro

```

## 初期開発

- AstroImportTree を開発する
- testdata/astro でテストを開発する(ava)
- テストがパスするように調整する
- README.md(英語)、README.ja.md(日本語)を作成する
- LICENSE を作成する(MIT)
- GitHub Acitons を用意する(Node.js 20/22/24 でテストが通ることを確認するシンプルなもの)

---
> Source: [ideamans/astro-import-tree](https://github.com/ideamans/astro-import-tree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
