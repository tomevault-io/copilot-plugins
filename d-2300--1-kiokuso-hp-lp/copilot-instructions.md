## 1-kiokuso-hp-lp

> 仙台市の店舗内装工事会社「記憶荘」のLP（ランディングページ）とHP（ホームページ）。

# 記憶荘（Kiokuso）LP/HP プロジェクト

## 概要
仙台市の店舗内装工事会社「記憶荘」のLP（ランディングページ）とHP（ホームページ）。
リポジトリ: kiokuso-hp-lp（GitHub）
本番: https://kiokuso.jp（Netlify自動デプロイ）

## 事業コンセプト（2026-07-05 ピボット: 繁盛コンセプト）
- **HP/LINE: 「繁盛する店は、物件選びから始まっている。」（D案）** — 元不動産の目利きで物件から伴走し、開業まで一気通貫
- **LP/広告の入口訴求は従来どおり具体物のみ**: 非公開テナント情報・実例・金額（伴走・繁盛の抽象語はLP禁止）
- ターゲット: 宮城県で店舗を始める個人オーナー・サラリーマン起業組・法人（初出店〜2・3店舗目）。飲食・美容・サロン中心
- 体制表現は**記憶荘中心**: 「窓口は元不動産出身の担当がひとりで一貫」＋「施工は記憶荘が実働の職人へ直接発注」。**「2人体制」「大工が中にいるから安い」「KEN個人の紹介」は使わない**（KENは協力会社的位置づけ）
- CTAはLINE友だち追加のみ（他のCTAは使わない）
- LINE公式（流入経路別URL）:
  - LP_tenant: lin.ee/9TamhEC（LP全般）
  - HP_top: lin.ee/UNm5XqPe（HP全般）
  - GBP: lin.ee/zJH3txi（Googleビジネスプロフィール用）
  - etc: lin.ee/94DJUIb（名刺・口コミ等オフライン）

## LP運用ルール（厳守）
- **LPでは「テナント探し」の具体物だけを打ち出す**。コンセプト哲学・世界観は出さない
- 以下の要素は**LPには入れない**（HP深部・LINE内・顧客化後のみ）:
  - 「一緒に勝つ」「並走」「伴走」系の抽象訴求
  - 「自走」「集客支援」「代理店卒業」系の啓蒙
  - AIパース・費用明細・提案書をメイン訴求にすること
- 「店舗内装 費用」「内装工事 費用」系の費用起点訴求は使わない（実顧客化しないことが判明）
- 「断ってOK」「営業なし」「相見積もり歓迎」は全LPで明示

## 技術スタック
- React + TypeScript + Vite + Tailwind CSS + React Router

## デプロイ
- mainブランチにpushでNetlify自動デプロイ
- 本番URL: https://kiokuso.jp

## 画像ルール（重要）
- ヒーロー画像以外は全て `-sm.webp`（480px幅）を直接参照する。srcSetは使わない
- 新しい画像追加時: `node scripts/resize-images.mjs 新画像.webp` で-sm版を生成
- コンポーネントで参照: `src="/images/filename-sm.webp"`
- ヒーロー画像（lp-hero.webp, group-hero-sakura.webp）のみオリジナル維持
- 一括生成: `node scripts/resize-images.mjs`（sharp依存、480px幅、WebP quality 80）
- ロゴ・アイコン（480px以下）はそのまま原本を参照

## デザイン方針（2026-07-05 デザインシステムF「黒を、引く。」に全面移行）
- 詳細規定: `docs/design-system-f.md`（HPは全ページこの体系・共通部品は `src/hp/components/FChrome.tsx`）
- 白 #FFFFFF が唯一の背景色。墨 #1A1A1A はテキストと細罫線。アクセントは深セージ #5C6C48 一色
- ベタ塗りでメリハリを出す: 番号チップ/最終CTA/見出しマーカー=セージ、KPI帯/フッター=墨
- 数字・金額・日付は Space Mono。番号付きセクション見出し（2桁ゼロ埋め＋上2px罫線＋日英併記）
- 写真は実写のみ。1pxの墨罫線で囲む。角丸・影は使わない
- **Yomogi等の手書きフォントは全面禁止**（2026-07-05「幼稚に見える」指示。LP含む）
- 写真主役・Before→After・値段先出しは継続。他社を貶める比較はしない（事実ベースの誠実さ）

## 運用チャネル
- Google広告（検索広告、アカウント: 183-797-5158）
- LINE公式アカウント（メインCV）
- GTM / GA4 / Microsoft Clarity でアクセス解析

## 作業ルール
- コミットメッセージは日本語OK
- 不明点があれば確認してから作業する（勝手に大きな構造変更をしない）
- 戦略・設計の詳細は `1-kiokuso/_context/strategy.md` を参照

---
> Source: [D-2300/1-kiokuso-hp-lp](https://github.com/D-2300/1-kiokuso-hp-lp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
