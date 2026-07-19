## html-slide-template

> HTMLスライドを効率的に生成・編集するための**複数テンプレート**プロジェクト。

# CLAUDE.md — HTMLスライドテンプレート集

HTMLスライドを効率的に生成・編集するための**複数テンプレート**プロジェクト。
**トークン節約が最重要の設計思想**: スライドは1枚=1HTMLファイルで独立しており、編集時は対象スライドだけを読めばよい。

## ディレクトリ構成

```
index.html                      テンプレート選択ページ(各テンプレートのビューアへのリンク)
templates/<name>/               1テンプレート=1フォルダ。それぞれ完全に自己完結
├── design-rules.md             そのテンプレートのデザイン定義(唯一の正)
├── layouts/                    レイアウトパターン雛形(日本語ダミーテキスト入り)。編集禁止のマスター
├── slides/                     実際のスライド(slide-01.html, ... ゼロ埋め連番)
├── slides.js                   スライドの一覧・順序(ビューアが参照)
└── index.html                  そのテンプレートのビューア+プレゼンモード
```

### 現在のテンプレート

| 名前 | デザイン |
|---|---|
| `Tech-Blue` | ダークネイビー+インディゴ→シアンのグラデーション。テック系・モダン |
| `Figma-Lite-Slides` | 白・黒・グレーのみのFigma風ミニマル(参照元: `04_Research/slide_sample/figma_01.pdf`) |
| `Figma-Dark-Slides` | 黒背景・白文字・グレーのみのFigma風ミニマル(参照元: `04_Research/slide_sample/figma_02.pdf`) |
| `Warm-Editorial` | クリーム地+錆色のマーカー下塗り強調。温かみのある雑誌風(雰囲気の参考: `04_Research/slide_sample/Tech_02.pdf`) |

## スライド作成の手順

1. **使うテンプレートを1つ決める**(ユーザー指定がなければ内容に合うものを提案)
2. そのテンプレートの `design-rules.md` を読む(初回のみ)
3. 内容に合うパターンを `templates/<name>/layouts/` から選び、`templates/<name>/slides/slide-NN.html` にコピー
4. ダミーテキストを実際の内容に置換(構造・スタイルは変えない。ダミー文が示す分量・行数の目安を守る)
5. `templates/<name>/slides.js` の `SLIDES` 配列にファイル名を追加

## 編集のルール(トークン節約)

- **既存スライドの修正時は、対象の `slides/slide-NN.html` だけを読む。** 他のスライドや index.html を読み直さない
- ページの追加・削除・並び替えは `slides.js` の配列編集で完結させる(削除時はファイルも消す)
- `layouts/` と各 `index.html` はマスターなので、ユーザーの明示的な指示がない限り編集しない
- テンプレート間でデザインを混ぜない(トークン・クラスはテンプレート内で完結)

## 新しいテンプレートを追加するとき

1. `templates/<new-name>/` を作り、`design-rules.md` にトークン・タイポグラフィ・余白・テキスト量上限を定義
2. `layouts/` に10〜14種の雛形、`slides/` にサンプル数枚、`slides.js`、ビューア `index.html`(既存テンプレートからコピーしてタイトルと配色だけ変更)を用意
3. ルート `index.html` のテンプレート一覧にブロックを1つ追加

## 確認方法

ルートの `index.html` をブラウザで開く → テンプレートを選ぶ → 全スライド一覧。クリックで拡大、`←/→` でページ送り、`F` でフルスクリーン、`Esc` で一覧に戻る。

---
> Source: [sakblm/html-slide-template](https://github.com/sakblm/html-slide-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
