## news

> https://github.com/sashibe/news.git

# MedBrief — CLAUDE.md

## リポジトリ

https://github.com/sashibe/news.git

## プロジェクト概要

**MedBrief** は医療従事者（主に歯科医師）向けの AI ニュースキュレーションサービス。
ユーザーが設定したキーワード・カテゴリをもとに、医療・防災関連の記事やプレスリリースを自動収集し、AI が要約・分析して毎朝メール or Slack に配信する。

**発注元:** 株式会社 AXP JAPAN（代表：Sori）  
**最初のターゲットユーザー:** 山田兄弟歯科グループ 山田先生（災害×医療が関心領域）  
**ビジネス文脈:** SmileCheck（AI 口腔自己チェックアプリ）のクライアントでもある。将来的に医療従事者向け SaaS へ展開を想定。

---

## 現在の状態

### 完成しているもの
- `frontend/` — v0 製 Next.js フロントエンド（Next.js 14 + TypeScript + shadcn/ui）
  - `app/page.tsx` — メインダッシュボード
  - `components/medbrief/` — Header / SettingsPanel / ArticleCard
  - **`generateBriefing` は現在ダミー実装（setTimeout + サンプルデータ）**

### Phase 1 でやること（最優先）
1. `app/page.tsx` の `generateBriefing` に Claude API を繋ぐ
2. Cloud Functions でキーワード → Claude → 記事生成（バックエンド）
3. 日次スケジューラー（毎朝 6:30）
4. SendGrid でメール配信
5. Firestore でキーワード設定を永続化

### 未実装
- Claude API 接続（フロント・バックエンド両方）
- 実際のニュースソース（Google News RSS、厚労省 RSS 等）
- メール・Slack 配信
- ユーザー認証（Firebase Auth）

---

## フロントエンド構成（v0 生成）

```
frontend/
├── app/
│   ├── page.tsx          # ★ メイン。generateBriefing を実装する
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── medbrief/
│   │   ├── article-card.tsx    # 記事カード（展開でAI分析表示）
│   │   ├── header.tsx          # ヘッダー（日付・受信者名）
│   │   └── settings-panel.tsx  # キーワード設定 UI
│   └── ui/                     # shadcn/ui コンポーネント群
├── package.json          # Next.js 14, TypeScript, shadcn/ui
└── pnpm-lock.yaml
```

### generateBriefing の現状（ダミー）

```typescript
// app/page.tsx
const generateBriefing = async () => {
  setIsLoading(true)
  await new Promise((resolve) => setTimeout(resolve, 1500)) // ← ダミー
  setArticles(SAMPLE_ARTICLES)  // ← ハードコードされたサンプルデータ
  setIsLoading(false)
}
```

### generateBriefing の実装目標

```typescript
const generateBriefing = async () => {
  setIsLoading(true)
  const res = await fetch("/api/briefing", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ keywords, recipientName }),
  })
  const data = await res.json()
  setArticles(data.articles)
  setIsLoading(false)
}
```

---

## バックエンド構成（未実装）

```
functions/
├── src/
│   ├── generateBriefing.js   # Claude API 呼び出し ← Phase 1 最優先
│   ├── collectNews.js        # RSS 収集
│   ├── sendEmail.js          # SendGrid 配信
│   └── scheduler.js          # Cloud Scheduler（毎朝 6:30）
└── package.json
```

---

## Anthropic API 呼び出しルール

```javascript
// モデルは必ずこれを使う
model: "claude-sonnet-4-6"

// web_search ツールはバックエンド（Node.js / Cloud Functions）からのみ使用可
// ブラウザ環境では動作しない
tools: [{ type: "web_search_20250305", name: "web_search" }]
```

### JSON レスポンスパターン

```javascript
system: "You are a Japanese medical news curator. Respond ONLY with a valid JSON object. No explanation, no markdown, no code fences. Start with { and end with }.",
messages: [{ role: "user", content: "..." }]
```

### JSON パースパターン（ロバスト版）

```javascript
const blocks = data.content || [];
const rawText = blocks.filter(b => b.type === "text").map(b => b.text).join("").trim();
const cleaned = rawText.replace(/^```json\s*/i, "").replace(/^```\s*/i, "").replace(/\s*```$/i, "").trim();
const start = cleaned.indexOf("{");
const end = cleaned.lastIndexOf("}");
const parsed = JSON.parse(cleaned.slice(start, end + 1));
```

---

## 記事データの型定義

```typescript
// article-card.tsx に既存の型と合わせること
interface Article {
  id: string;
  headline: string;      // 見出し
  category: string;      // カテゴリ（例: 災害医療、歯科防災、制度・政策）
  date: string;          // 掲載日または時期
  summary: string;       // 要約（100文字以内）
  aiAnalysis: string;    // 歯科医師・医療従事者への示唆（150文字以内）
  source?: string;       // 情報源名
  url?: string;          // 元記事URL
}
```

---

## プロンプト設計

```
あなたは医療・防災専門のニュースキュレーターです。
対象読者は歯科医師・歯科医院経営者です。
以下の観点で分析してください：
1. 歯科医院の BCP（事業継続計画）への影響
2. 患者への口腔ケア指導への活用可能性
3. 医療従事者として知っておくべき制度・政策変更
```

---

## 技術スタック

| レイヤー | 技術 |
|--------|------|
| フロント | Next.js 14 / TypeScript / shadcn/ui / Tailwind |
| バックエンド | Firebase Cloud Functions (Node.js) |
| DB | Firestore |
| 認証 | Firebase Auth（Phase 2） |
| AI | Anthropic API（claude-sonnet-4-6） |
| メール | SendGrid |
| Slack | Incoming Webhook |

---

## 開発優先順位

### Phase 1（MVP / 山田先生向け）
1. `app/api/briefing/route.ts` を作成 → Claude API を呼ぶ Next.js API Route
2. `generateBriefing` を API Route に繋ぐ
3. Cloud Functions で日次スケジューラー（毎朝 6:30）
4. SendGrid でメール配信
5. Firestore でキーワード設定を保存

### Phase 2（SaaS 化）
1. Firebase Auth でユーザー登録・ログイン
2. 配信履歴ダッシュボード
3. Slack 配信オプション
4. 複数ユーザー対応

### Phase 3（拡張）
1. RSS 接続（厚労省・内閣府防災・日本歯科医師会）
2. PubMed 論文要約
3. 重要度スコアリング

---

## コーディング規約

- **言語:** TypeScript（フロント）/ JavaScript（Cloud Functions）
- **コミット:** `[phaseN] 変更内容`（例: `[phase1] Claude API route 追加`）
- **エラー処理:** ユーザー向けには日本語メッセージ
- **API キー:** Firebase Secret Manager。コードへのハードコード厳禁

---

## 関連プロジェクト（AXP JAPAN）

| プロジェクト | 概要 | 共有できる資産 |
|------------|------|--------------|
| Cocomi | 予測市場アプリ | Firebase 構成、Claude API パターン、RSS→Claude パイプライン |
| SmileCheck | AI 口腔チェックアプリ | 山田先生との関係性、歯科ドメイン知識 |
| SAKURA | 日経225先物自動売買 | Claude Code 運用パターン |

---

## 引き継ぎメモ

- デモ実施: 2026年4月16日 山田先生との打ち合わせ
- v0 でフロントエンドを生成。`generateBriefing` のみダミー実装のため、**Phase 1 の最初の仕事は API Route の実装**
- Firebase プロジェクトは新規作成。Cocomi とは別プロジェクトで管理
- Cocomi の RSS → Claude → Telegram パイプラインのアーキテクチャを参考にすること
- デザイントーン: ネイビー × ティール（v0 が忠実に再現済み）

---

*最終更新: 2026-04-16 / Sori（AXP JAPAN）*

---
> Source: [sashibe/news](https://github.com/sashibe/news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
