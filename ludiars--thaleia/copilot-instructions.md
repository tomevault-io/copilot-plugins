## thaleia

> MUSA「9 女神」の **タレイア** = 企画↔実装トレーサビリティ。

# Thaleia — Claude Code ルール

## 性格

MUSA「9 女神」の **タレイア** = 企画↔実装トレーサビリティ。
**Praeforma(人間の仕様) と Anatomia(コード解析) の両方と会話してデータを統合する単独サービス**。

役割分担を崩さない:
- **Anatomia** = 解析情報 / LLM データキャッシュ (機械向け)。仕様的概念は持たない。
- **Praeforma** = 人間が理解する仕様/ドメイン/要件の正本。
- **Thaleia** = 上記 2 つを **突合** して統合ビューを作る。自分で解析や仕様編集はしない
  (両サービスを呼ぶだけ。エンジンを二重実装しない)。

## ブランチ + PR 運用

すべての変更は feat ブランチ → PR → squash merge。main 直 push 禁止。AI 実装は 1 PR に集約。

## コード規約

共通規約は `coding-conventions` スキル (= AIFormat/RULE_CODE.md) を正本とする。固有の追加:

- **無言フォールバック禁止**: 上流 URL/token 未設定や上流エラーは mock に落とさず明示エラー。
- **SRP**: `clients/`(上流取得) / `reconcile/`(突合ロジック) / `routes/`(HTTP) / `store/`(永続化)
  を分離。`reconcile/match.ts` は**純粋関数**に保つ (上流 I/O を持ち込まない → fixture テスト可)。
- ESM (`"type":"module"`)、import は `.ts` 拡張子付き (tsx + NodeNext)。

## ビルドとテスト

```sh
npm run typecheck   # tsc --noEmit
npm test            # vitest run
npm start           # tsx src/index.ts (dev は npm run dev)
```

PR 前に typecheck green + test 全通。

## spec/

AIFormat の分類フォルダに揃える (`data` / `feature` / `interface` / `setup` / `test`)。
- `spec/thaleia.md` — 全体設計
- `spec/feature/reconcile.md` — 突合アルゴリズム
- `spec/interface/api.md` — REST + 上流クライアント契約

## env

| 変数 | 既定 | 役割 |
|---|---|---|
| `THALEIA_PORT` | 8890 | HTTP ポート (PORT-MAP へ登録予定) |
| `PRAEFORMA_BASE_URL` | (なし) | Praeforma API。未設定なら /reconcile は 503 |
| `PRAEFORMA_TOKEN` | (なし) | Praeforma 用 Cernere service token |
| `ANATOMIA_BASE_URL` | (なし) | `anatomia web` の URL。未設定なら /reconcile は 503 |
| `PRAEFORMA_WEB_URL` | (なし) | ビューアの Praeforma ディープリンク先。未設定ならリンク無効表示 |
| `ANATOMIA_WEB_URL` | `ANATOMIA_BASE_URL` | ビューアの Anatomia ディープリンク先 (未設定なら API base 流用) |
| `THALEIA_LLM_RANK` | (なし) | `1` で `/relay/anatomia` の LLM 意味ランキングを有効化 (既定 off=構造選択のみ) |
| `THALEIA_SEMANTIC_MATCH` | (なし) | `1` で reconcile の意味照合を有効化 (字面で外れた未対応を `claude -p` で対応付け。既定 off) |
| `THALEIA_CLAUDE_BIN` | `claude` | LLM 用 claude CLI (Windows は git-bash wrapper を差せる) |
| `THALEIA_RANK_TOP_K` | `40` | ランキング後に残す上限件数 |
| `THALEIA_LINK_CONFIDENCE_THRESHOLD` | `0.7` | spec-link ワークリストで「低 confidence」とみなす閾値 (この値未満を要レビュー)。 0..1 外/不正値は既定に倒す |

## 完了済 (主要機能)

- 突合 v1 (domain) + scene(layout) target (二部マッチで layout↔screen)。
- MUSA リレー `/relay/anatomia` + LLM 意味ランキング (opt-in)。
- 突合履歴の差分 (前回比 diff)。
- 関数粒度トレース `/api/trace` (spec-links を関数へ join)。
- spec 単位トレース `/api/trace/spec` (Pf spec → target ドメインの実装関数を直結。 viewer の仕様カバレッジ行「関数 ▾」)。
- 未批准 spec-link ワークリスト `/api/links/worklist` (ratified:false / 低 confidence を要レビューで列挙。 閾値 `THALEIA_LINK_CONFIDENCE_THRESHOLD` 既定 0.7。 viewer に専用セクション)。
- 意味照合 (opt-in、字面で外れた未対応を `claude -p` で対応付け = 真 embedding の代替)。
- 実環境フル E2E 通済 (studio anatomia-link / scene / diff / trace / semantic を実 Pf+An で確認)。

## 次フェーズ

- Praeforma フロントが `?tab/focus` クエリを消費して個別エンティティへ着地 (deeplink は対応済)。
- spec-link ワークリストから直接 Anatomia の批准 UI へディープリンク (現状は表示のみ)。

---
> Source: [LUDIARS/Thaleia](https://github.com/LUDIARS/Thaleia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
