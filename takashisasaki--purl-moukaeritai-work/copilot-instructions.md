## purl-moukaeritai-work

> このリポジトリは GitHub Pages 上で `purl.moukaeritai.work` として公開される。

# Repository Rules

このリポジトリは GitHub Pages 上で `purl.moukaeritai.work` として公開される。  
このリポジトリ内のコンテンツがリダイレクトの受け皿になるのは `/gag`、`/get-a-grip`、`/ts` のみである。  
トップページは `purl.org` への案内兼、入力したパスから `purl.org` 側の行き先を調べるためのページとして扱う。  
`/search.html` は `archive.org` の Wayback API を使って、`purl.org` と `purl.archive.org` の保存状況を調べるための専用ページとして扱う。  
それ以外のリダイレクト先にアクセスしたい場合は直接 `purl.org` を案内する。

## Attribution

- 作者名は `Takashi Sasaki` である。
- 作者のホームページは `https://x.com/TakashiSasaki` である。
- 各ページのフッタには、開発者名と `x.com/TakashiSasaki` を載せる。

## Social Preview

- トップページ `/` には Open Graph と Twitter Card のメタデータを置く。
- カード画像は `twitter-card.png` を使う。
- カード文言は `moukaeritai.work` の永続識別子ランディングページであることが伝わる内容にする。

## Cache Policy

- HTML ページには `Cache-Control: no-cache, max-age=600, must-revalidate` を示す `meta http-equiv` を入れる。
- 開発中は HTML や SNS カード画像の変更が 10 分以内に見直されることを優先する。
- GitHub Pages の配信ヘッダ自体はこのリポジトリからは制御できない前提で考える。

## Routing Policy

- トップページ `/` の canonical URL は `https://purl.moukaeritai.work/` である。
- `/` は `purl.org` への案内とパス調査のページとして扱う。
- `/search.html` は Wayback API を使ったパス検索ページとして扱う。
- `/gag`、`/get-a-grip`、`/ts` は、それぞれ対応する `purl.org` 側の永続URLの案内ページとして扱う。
- それ以外の存在しないパスは `404.html` を表示する。

## 404 Handling

- `404.html` は JavaScript で現在の `location.pathname`、`location.search`、`location.hash` を読み取り、`https://purl.org` をオーソリティにした候補URLを組み立てて表示する。
- 404 画面では、その候補URLをリンクとして見せ、コピーしやすい形にする。
- `purl.archive.org` 由来のリクエストでも、案内先は `purl.org` を正とする。

## Maintenance Notes

- このサイトには実質的なコンテンツを増やさず、リダイレクトと案内に集中する。
- ルーティング方針を変えるときは、`index.html`、各サブディレクトリの `index.html`、`404.html` をまとめて見直す。
- 仕様が増えたら、このファイルに追記して次の作業者が同じ前提で動けるようにする。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TakashiSasaki)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TakashiSasaki)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
