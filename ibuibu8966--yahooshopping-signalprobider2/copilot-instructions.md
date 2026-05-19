## yahooshopping-signalprobider2

> ヤフーショッピングを活用した自動せどりサービスのランディングページ。

# センリツヤフショシグナルプロバイダー LP

## プロジェクト概要

ヤフーショッピングを活用した自動せどりサービスのランディングページ。
オーナーは資金を投下するだけで、アカウント運用から買取までのすべてを代行し、月額費以上の利益を保証するフルマネージドサービス。

## 技術スタック

- **フレームワーク**: Next.js (App Router)
- **言語**: TypeScript
- **スタイリング**: Tailwind CSS
- **UIライブラリ**: shadcn/ui または自作コンポーネント
- **アニメーション**: Framer Motion（オプション）

## ディレクトリ構成

```
senritsu-lp/
├── app/
│   ├── page.tsx          # メインLP
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── Header.tsx        # 固定ナビゲーション
│   ├── Hero.tsx          # ヒーローエリア
│   ├── Partners.tsx      # 導入企業・パートナーロゴ
│   ├── Benefits.tsx      # ベネフィットセクション
│   ├── Steps.tsx         # 導入手順（3ステップ）
│   ├── Pricing.tsx       # 料金表セクション
│   ├── Testimonials.tsx  # お客様の声
│   ├── Stats.tsx         # 評価実績
│   ├── FAQ.tsx           # よくある質問
│   ├── CTA.tsx           # 最終CTAセクション
│   └── Footer.tsx        # フッター
├── public/
│   └── images/
└── lib/
```

## デザイン要件

### カラーパレット

| カラー名 | カラーコード | 用途 |
|----------|-------------|------|
| ダークネイビー | `#0f0f23` | メイン背景、ヘッダー |
| ゴールド | `#D4AF37` | アクセント、CTA、アイコン |
| ヤフショレッド | `#FF0033` | ポイント強調、価格表示 |
| ホワイト | `#FFFFFF` | テキスト、カードベース |
| ライトグレー | `#F5F5F5` | セクション背景（交互） |

### タイポグラフィ

- **見出し**: Noto Sans JP Bold / 游ゴシック
- **本文**: Noto Sans JP Regular
- **数字・価格**: Inter / Roboto Mono（等幅）

### レスポンシブ

- モバイルファースト設計
- ブレークポイント: 640px / 768px / 1024px / 1280px

## 料金体系

| 項目 | 料金（税込） |
|------|-------------|
| 初期費用 | 330,000円 |
| 月額費用 | 110,000円 |

## 会社情報

- **会社名**: 株式会社SENRITSU
- **代表取締役**: 加藤 大成
- **所在地**: 〒579-8036 大阪府東大阪市鷹殿町11-2 カワショウビル2・3階

## 開発コマンド

```bash
npm run dev      # 開発サーバー起動
npm run build    # ビルド
npm run start    # 本番サーバー起動
npm run lint     # リント実行
```

## セクション構成

1. 固定ナビゲーション（Header）
2. ヒーローエリア（Hero）
3. 導入企業・パートナーロゴ（Partners）
4. ベネフィットセクション（Benefits）
5. 導入手順（Steps）
6. 料金表セクション（Pricing）
7. お客様の声（Testimonials）
8. 評価実績（Stats）
9. FAQ（FAQ）
10. 最終CTAセクション（CTA）
11. フッター（Footer）

---
> Source: [ibuibu8966/yahooshopping-signalprobider2](https://github.com/ibuibu8966/yahooshopping-signalprobider2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
