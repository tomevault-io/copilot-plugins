## exam-venue-guide

> 試験会場ガイド（*-center / exam-venue-and-region）の執筆・修復ルール


# 試験会場ガイド記事

**正本:** `docs/exam-venue-guide.md`

## 禁止（本文・FAQ）

- 会場の**具体住所**、**駅名・バス停名**、**徒歩・乗換ルート**、**出口・フロア番号**をサイト独自に記載しない
- 二次情報（まとめサイト・口コミ）の交通案内を転載しない

## 必須

- 正本は **受験票** + **公式会場案内**（リンクで誘導）
- `primary_sources` に公式会場 URL を含める
- 本文は `guide_content_shared` の `exam_venue_*` / `jissh_center_list_prose` 経由（lib は `patch_guide_venue_official_links.py`）
- URL は `tools/exam_venue_official_links.py`（JISSH）または `site-config.json` の `externalLinks`

## 対象 slug

- `*-center` … センター個別 → `fix_exam_venue_guide_articles.py`
- `exam-venue-and-region` / `shiken-kaijo` … ハブ → `fix_exam_venue_hub_articles.py`

## 作業後

```bash
python3 tools/fix_exam_venue_guide_articles.py --target <サイト>
python3 tools/fix_exam_venue_hub_articles.py --target <サイト>
python3 tools/build_article_pages.py
```

全サイト: `bash tools/run_exam_venue_official_links_batch.sh`

アクセス節の箇条書きは「確認項目」まで。駅名・住所の**値**は読者が公式ページで確認する前提。

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
