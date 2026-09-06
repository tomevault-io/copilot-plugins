## guide-prose-quality

> ガイド記事 prose の具体性・自然さ（抽象表現・メタ確認・FAQ 定型の禁止）


# ガイド prose 品質ルール

**正本:** `tools/guide_prose_patterns.py`（検出） / `tools/guide_content_shared.py`（生成）

## 禁止パターン（読者向け本文）

| 種別 | 悪い例 | 対応 |
|------|--------|------|
| メタ確認だけ | 「〇〇」の詳細は公式の最新要項で確認してください | 具体手順・箇条書きまで書く |
| 読了目的の抽象 | 行動チェックリストに沿って… / 確認すべき点と演習・用語解説… | `user_intent_prose()` |
| FAQ 定型 | 条文や指針の主体（事業者・…）をセットで | `faq_official_verify_answer()` |
| FAQ 内部記法 | `「質問」→ 答え` / `FAQ3「」→` | 「については、」に言い換え |
| フォールバック崩れ | `…の要点を。`（読点分割で述語欠落） | `keyword_fallback_default` の「要点は」形式 |
| 行動項目の空疎 | 混同語を1周 / 分野タグ付き10問（だけ） | `action_items_prose()` |

## 書き切り必須のテーマ

- **持ち物・チェックリスト・当日の流れ・アクセス** → `guide_content_shared` の `exam_day_*` / `exam_venue_access_prose` を使い、`- ` 箇条書きまで書く（括弧だけの列挙禁止）。
- **会場ガイド**（`*-center` 等）ではアクセス箇条書きに**具体の駅名・住所を書かない**。詳細は `.cursor/rules/exam-venue-guide.mdc`。

## 節の最小文字数（180字）

- `ensure_min` の filler にメタ確認1文だけを使わない。
- `section_body_min_filler(heading, topic, official)` で「テキスト該当章＋演習＋要項照合」の具体1文を足す。

## 修正パイプライン（全サイト）

```bash
# exam-site-shell 正本で lib パッチ後
python3 tools/patch_guide_prose_templates.py
bash tools/run_guide_prose_batch.sh
python3 tools/audit_guide_prose_quality.py --root ~/Projects/eisei2shu-master --strict
```

1. `fix_guide_duplicate_bodies.py --all-published`
2. `fix_guide_prose_quality.py`
3. `fix_editorial_auto.py`
4. `build_article_pages.py`
5. `audit_guide_prose_quality.py --strict`

## 監査

- CSV: `scan_prose_text()` — `tail_section_ref`, `vague_user_intent`, `broken_fallback` 等
- HTML: slug メタ・節番号崩れ・field_boiler

**PASS 条件:** CSV/HTML とも hits=0。`public_site/` は `.gitignore` のため配信前に `build_all.py` 必須。

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
