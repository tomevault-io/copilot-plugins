## content-authoring

> SEO articles, glossary, editorial quality, diagrams, affiliate


# コンテンツ執筆（要約）

**正本:** `docs/seo-article-guidelines.md`（記事・用語・色・公開境界）

| 種別 | ドキュメント | スクリプト |
|------|-------------|------------|
| 試験ガイド | `guide-article-template.md` | `scaffold_guide_article.py` |
| 用語 | `glossary-term-template.md` | `scaffold_glossary_term.py` |
| アフィリエイト | `affiliate/multi-site-affiliate-workflow.md` | `scaffold_affiliate_article.py` |
| 図解 | `term-diagrams.md` | `term_diagram.py` |

知識ハブ（比較・数値・誤答タブ）は**廃止**。旧 URL は `build_hub_retire_redirects.py` でリダイレクト。

編集後は必ず `python3 tools/build_all.py`。

ガイド記事の抽象表現・メタ確認だけの節・FAQ 定型は `.cursor/rules/guide-prose-quality.mdc` を参照。修復は `fix_guide_prose_quality.py` と `run_guide_prose_batch.sh`。

**試験会場ガイド**（`*-center` / `exam-venue-and-region` / `shiken-kaijo`）は `.cursor/rules/exam-venue-guide.mdc` と `docs/exam-venue-guide.md` に従う。住所・駅名・ルートは書かず公式リンクのみ。修復は `fix_exam_venue_guide_articles.py` / `fix_exam_venue_hub_articles.py`。

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
