## knowledge-hub-content

> Knowledge hub compare/numbers/mistakes — 150 per type, CSV quality


# Knowledge Hub Content（比較・数値・誤答）

**正本:** [docs/knowledge-hub-article-templates.md](../docs/knowledge-hub-article-templates.md)

## 本番目標（各 CSV 独立）

| 種別 | CSV | 目標 |
|------|-----|------|
| 比較・整理表 | `comparisons.csv` | **150〜153 件** |
| 数値・期限早見表 | `numbers.csv` | **150〜153 件** |
| よくある誤答 | `mistakes.csv` | **150〜153 件** |

150 件未満 → `validate_csv.py` **WARN**（本番目標）。件数超過は更新負荷の WARN のみ。

## 新規作成（必須フロー）

```bash
python3 tools/scaffold_knowledge_hub_article.py --type compare|numbers|mistakes \
  --title "…" --category "…" --append
# プレースホルダをすべて差し替え
python3 tools/validate_csv.py
python3 tools/build_glossary_pages.py
```

## 1記事 = 1検索意図

- 「AとBの違い」→ compare
- 「〇〇 何日 何%」→ numbers
- 「〇〇 間違えやすい」→ mistakes

用語解説に比較表を詰め込まず、混同語は **compare / mistakes へ切り出す**。

## 行品質（ERROR）

- JSON 列（`compare_rows` / `item_rows` / `pattern_rows`）が有効
- `related_terms` … **glossary に登録済み 2 件以上**
- FAQ **4 件**、`exam_points` **2 項目以上**
- 雛形マーカー（`【記入】` 等）なし

## 禁止

- 用語解説への比較表の丸ごと複製
- 3種 CSV を手書き HTML で代替
- 件数目標なしのサンプル行だけで本番公開

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
