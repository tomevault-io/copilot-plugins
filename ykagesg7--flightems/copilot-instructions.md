## flightems

> PPL/CPL MDX記事の作成・編集時に適用するコンテンツガイドライン


# MDX 記事作成ガイドライン

## メタデータ形式（必須）

YAML frontmatter (`---` ブロック) は使用禁止。ESM export 形式のみ対応。

```mdx
import { Highlight, Image } from '@mdx'
import React from 'react'

export const meta = {
  type: 'lesson',
  title: '【科目名】記事タイトル ～サブタイトル～',
  slug: 'ppl-1-1-1-topic-name',
  date: '2026-01-01',
  author: 'Michizane (Shadow)',
  tags: ['PPL', '学科試験', '科目名', 'トピック'],
  series: 'PPL-Master-Syllabus',
  order: 1,
  readingTime: 12,
  excerpt: '記事の要約（100〜200文字）',
  publishedAt: '2026-01-01'
};
```

### 必須フィールド
- `title` (string): 記事タイトル
- `slug` (string): URL スラッグ（一意）
- `tags` (string[]): タグ配列
- `type` ('article' | 'lesson'): PPL記事は `'lesson'`

### 推奨フィールド
- `series` (string): シリーズ名 — PPL記事は `'PPL-Master-Syllabus'`
- `order` (number): シリーズ内の順序番号
- `readingTime` (number): 読了時間（分）
- `excerpt` (string): 要約
- `publishedAt` (string): 公開日 YYYY-MM-DD
- `author` (string): 著者名

### 禁止事項
- YAML frontmatter (`---` で囲むブロック) を使用しないこと
- `remark-frontmatter` プラグインは未導入のため、YAML は通常テキストとして表示されてしまう
- メタデータは `articlesIndex.ts` が `module.meta` として読み取る ESM export のみ対応

## ファイル命名規則

PPL記事: `PPL-{Subject}-{Section}-{Topic}_{TitleInPascalCase}.mdx`

例:
- `PPL-1-1-1_TemperatureBasics.mdx` (Subject 1, Section 1, Topic 1)
- `PPL-1-2-5_CarburetorMixture.mdx` (Subject 1, Section 2, Topic 5)
- `PPL-2-1-1_CloudTypes.mdx` (Subject 2, Section 1, Topic 1)

Subject 番号:
1. 航空工学
2. 航空気象
3. 空中航法
4. 航空通信
5. 航空法規

## 画像記法

全ての画像に `<Image>` コンポーネントを使用する。`<img>` タグや `![alt](url)` マークダウン記法は禁止。

### ヒーロー画像（記事冒頭）
```mdx
<Image
  src="https://placehold.co/1200x630/1a1a1a/FFF?text=Image+Title"
  alt="画像の説明"
  width={1200}
  height={630}
/>
```

### アフィリエイト枠内の画像
```mdx
<Image
  src="https://placehold.co/300x300/color/text?text=Product+Name"
  alt="商品名"
  width={300}
  height={300}
  className="rounded-md w-full"
/>
```

## 記事構成（標準セクション順序）

1. **import文 + export const meta** — メタデータ
2. **ヒーロー画像** — `<Image>` コンポーネント
3. **🎯 今日のミッション（学習目標）** — チェックリスト形式（3項目）
4. **📋 概要** — トピック導入、重要性の説明
5. **📚 詳細解説** — メインコンテンツ（サブトピック1, 2, ...）
6. **🍵/🍜/🥃 五感で覚える航空力学：給油ポイント** — 福岡名物に例えた比喩解説 + アフィリエイト枠
7. **💡 試験対策：ここが出るばい！（Check Six）** — 敵機パターン形式の問題解説
8. **📝 まとめ：道真のフライトデブリーフィング** — 3点要約 + 次回予告
9. **export default** — `export default ({ children }) => <>{children}</>`

## キャラクター・口調ガイドライン

### 道真（Michizane / Shadow）のペルソナ
- 博多弁（筑前方言）を基調とした語り口
- 航空の専門知識を日常の比喩で伝える教官キャラ
- ユーモアと厳しさのバランス

### 頻出表現
- 語尾: 「〜たい」「〜ばい」「〜けん」「〜よ」「〜んよ」
- 呼びかけ: 「おい、そこのあんさん」「あんさん」
- 感嘆: 「ほら」「ばってん」「〜やろ？」
- 試験: 「敵機パターン」「Check Six」

### 比喩の原則
- 福岡・九州の食文化や名所を優先（八女茶、博多うどん、通りもん、もつ鍋、豚骨ラーメン、糸島、中洲 等）
- 抽象的な航空概念を具体的な五感体験に変換する
- 各記事で異なる名物を使い、シリーズ全体で多様性を持たせる

## 数式

KaTeX 記法を使用:
- インライン: `$F = 1.8 \times C + 32$`
- ブロック: `$$ n = \frac{揚力}{重力} $$`

## DOMネスト回避（validateDOMNesting 対策）

MDX パース時に `<p>` 内にブロック要素がネストされると `validateDOMNesting` 警告が発生する。以下を守ること。

### 禁止事項
- アフィリエイト枠（`<div className="flex ...">`）の**直前に** JSX コメント（`{/* ... */}`）を置かない
- 段落内に `<div>`, `<blockquote>`, `<ul>`, `<ol>` 等のブロック要素を直接ネストしない

### 推奨
- アフィリエイト枠の前は空行で区切り、直前の段落と明確に分離する
- コメントが必要な場合は、アフィリエイト枠の外側（前のセクション末尾など）に配置する
- `MDXContent.tsx` の `p` コンポーネントは `<div>` でレンダリングする設計のため、通常の段落は問題ない

## テンプレート

新規記事作成時は `docs/templates/PPL_Article_Template.mdx` を参照すること。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ykagesg7)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ykagesg7)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
