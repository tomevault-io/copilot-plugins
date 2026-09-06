## practice-ichimon-static

> 実践演習・一問一答の静的 q/ ページ・CSV・一覧 UI の契約


# Practice / Ichimon Static Pages

**正本:** `docs/question-static-pages.md` · **統合:** `docs/integration-checklist.md` §1.5, §6
**検証:** `validate_site_integration.py`（CSV 件数・`#q-index-config` 契約）

## データの正本

| 種別 | CSV | 生成先 | SPA JS |
|------|-----|--------|--------|
| 実践 | `data/practice_questions.csv` | `q/practice/` | `exam-site-data-practice.js` |
| 一問一答 | `data/ichimon_questions.csv` | `q/ichimon/` | `exam-site-data-ichimondou.js` |

**`q/` 配下 HTML は手編集しない。** CSV または取り込みスクリプト → `build_all.py`。

## 一覧 UI 契約（`INDEX_CONFIG` → `#q-index-config`）

| 項目 | 実践 | 一問一答 |
|------|------|----------|
| `groupBy` | `category` | `category`（**年度別にしない**） |
| グループ順 | `site-config.json` の `fields` 順（`categoryOrder`） | 同左 |
| グループ内 | `question_no` 昇順 | `id` 昇順 |
| `statusFilters` | `wrong`, `bookmark` のみ | 同左 |
| 表行 | `build_practice_index_table_row` | `build_ichimon_index_table_row` |

過去問の `?status=exempt` 等は `sanitizeFiltersFromUrl()` で無効化されるが、**statusFilters に載せない**のが正。

## 大量取り込み（SPA バンク → CSV）

宅建など `ORIG_QUESTIONS` だけ充実している場合:

```bash
python3 tools/import_orig_questions_to_practice_csv.py   # takken-data-original.js が必要
python3 tools/import_base_questions_to_ichimon_csv.py --keep-manual
python3 tools/build_all.py
```

- 一問一答自動生成は過去問＋実践の4択から ○× 1 文（個数・組合せは除外）
- 手書き行は `source` に `auto-import` を含めない → `--keep-manual` で先頭保持

## 作業手順（サイト1つ）

1. **テンプレ**で generator / `site-q-index.js` / 取り込みスクリプトを直す
2. `python3 tools/build_all.py`（テンプレで成功確認）
3. 本番: `sync_from_template.py`（フル or サイト `manifest-phase*.txt`）→ 本番 `data/` は維持
4. 本番で取り込み（必要時）→ **`build_all.py`**
5. `validate_site_integration.py` + `validate_internal_links.py` → commit / push

## 禁止

| 禁止 | 理由 |
|------|------|
| 一問一答一覧を `groupBy: year` に戻す | ユーザー要望は分野順。URL の `y{年}` は id 由来で維持 |
| 生成 `q/practice/` を手修正 | 次回ビルドで消える |
| CSV 未更新のまま SPA JS だけ1000問 | 一覧 `#q-index-data` と CSV が不一致 |
| 宅建へフル sync で `build_past` 上書き | `sites/takken-master/SITE.md` 参照 |

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
