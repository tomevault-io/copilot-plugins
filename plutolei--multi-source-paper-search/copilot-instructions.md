## multi-source-paper-search

> >-


# Multi-Source Paper Search

Turn one request into a cross-source literature scan. The hard part isn't
hitting four APIs — it's reconciling four differently-shaped result sets into a
single honest picture: what's corroborated, what's contested, what only one
source saw. This skill splits that cleanly:

- **Deterministic work → `scripts/fanout.py`.** Fan-out, normalization, and
  paper-level dedup (DOI / arXiv-id / title via union-find) are mechanical and
  must be exact. The script does them and emits one JSON object. Don't eyeball
  dedup yourself — it's unreliable and burns tokens.
- **Semantic work → you.** Query expansion, claim-level cross-validation, and
  writing the cited report need judgement. That's your half.

## Procedure

### 1. Expand the query (light)
The academic sources are English-dominant, so translate a non-English request
to English, then write **up to 2** extra reformulations using method aliases /
synonyms (e.g. "RAG" ↔ "retrieval-augmented generation"). More than ~3 total
variants rarely adds recall and just costs API calls. Dedup across variants is
free — the script handles it — so variants are cheap insurance for recall.

### 2. Fan out + align (run the script)
```bash
python scripts/fanout.py --depth 20 "variant 1" "variant 2" "variant 3"
```
If the user has a project `.env` with keys (e.g. `OPENROUTER_API_KEY`), load it
first: `set -a && source .env && set +a`. The script auto-probes every source
and skips any with a missing key/script — it never fails the run; OpenAlex needs
no key and is always live. Read the JSON it returns: `papers` (deduped, ranked,
each with `consensus` = # distinct sources and `per_source_unique`),
`sources_used`, `skipped_sources`, and `perplexity_narrative`.

### 3. Cross-validate claims (semantic, lazy)
From the top deduped papers' abstracts and the `perplexity_narrative`, pull the
key claims / numbers / conclusions. Compare across sources: what's echoed by
multiple sources (→ consensus), what conflicts (→ divergence). Treat the
Perplexity narrative as a lead to corroborate, **not** as settled truth.
Only when the top-3 highest-consensus / highest-cited papers actually disagree
on a claim, fetch those few papers' full text to adjudicate — arXiv hits carry
full text in the `content` they were built from; otherwise open the `url`. Don't
pull full text by default; it's expensive and usually unnecessary.

### 4. Write the report
Follow `references/report-template.md` exactly: 检索概况 → 共识发现 → 分歧高亮
→ 每源独有贡献 → 论文排名表 → 参考文献. Match the user's language. Plain
markdown, no charts (the reader pastes into Feishu). Always state "used X / 4
sources" so consensus counts are read in the right context. Be honest when
recall is thin or sources were skipped — don't pad.

### 5. Offer to save
Print the report in the conversation first.
🔴 **CHECKPOINT — do NOT write any file unprompted.** Ask whether to save and
where. Save by default is wrong here: the user usually just copies the markdown.
Only after they say yes, write to the path they give.

## Failure modes & fallbacks
Don't assume the happy path. Each row is `trigger → first fix → if still stuck`:

| 触发条件 | 一线修复 | 仍失败兜底 |
|---|---|---|
| `fanout.py` 返回 `papers: []`（0 召回） | 放宽 query（去专有缩写）/ 降 `--depth` / 多给 1 个变体重跑 | 报告如实写"未召回"，建议换检索术语；不要编造论文 |
| `skipped_sources` 只剩 OpenAlex 一源 | 报告照出，但**显式标注"单源，无交叉验证价值"** | 提示用户补 key（见 `references/sources.md`）后重跑 |
| `perplexity_narrative: []`（key 失效/余额耗尽） | claim 级核验改用 OpenAlex/S2/arXiv 摘要，报告标注"无 Perplexity 叙事" | 提示查 OpenRouter 余额；其余三源仍出报告 |
| arXiv 报 JSON/`Unterminated string` | 已由 `fanout.py` 临时文件兜底（Node pipe-flush）；正常应不再出现 | 仍失败则该源进 `skipped_sources`，报告标 3/4 源 |
| 所有 `consensus` 全 = 1（各源不重叠） | **这不是 bug**：各源 top-N 正交。改走 claim 级共识 | 报告"每源独有贡献"段说明覆盖正交，不硬凑 paper 级 consensus |
| `fanout.py` 直接报错/无法运行 | 检查 python 可用、`.env` 是否在 skill 根目录 | 报告失败原因给用户，不静默改用单源手搜冒充多源 |

## Sources & keys
See `references/sources.md` for each source's invocation, key/env, the
degradation contract, and how to add a fifth source. Quick map: OpenAlex (no
key) · Semantic Scholar (optional `S2_API_KEY`) · arXiv (Valyu key via
arxiv-search setup) · Perplexity (`OPENROUTER_API_KEY`).

## Anti-patterns — do NOT do these
The whole value is honest cross-source reconciliation. These break it:

- ❌ **肉眼去重 / 手动合并论文** — 用 `fanout.py` 的确定性去重（DOI/arXiv-id/标题并查集）；人工去重不可靠且烧 token。
- ❌ **只跑一个源冒充多源** — 单源出的报告没有 consensus 意义，必须标注降级，不能包装成"多源已验证"。
- ❌ **把 Perplexity 叙事当已证结论** — 它是 web 合成的线索，需被其他源印证才进"共识"；只它提的进"分歧/独有"。
- ❌ **默认拉全文 / 默认存文件 / 默认出图** — 全文仅 top-3 冲突时拉；存盘要问；报告用 markdown 表不出图。
- ❌ **把 3 源跑出的 consensus 当 4 源口径** — consensus 是相对已跑源数的，2/2 弱于 2/4，概况里讲清。
- ❌ **粉饰稀疏召回** — 召回薄、源跳过多、共识弱时如实写，不堆砌、不补论文凑数。

## Notes
- Depth default is 20/source ("standard"). Drop to ~10 for a quick scan, raise
  to 30+ for an exhaustive one.
- `consensus` is relative to the sources that actually ran — a 2 out of 2 live
  sources is weaker corroboration than 2 out of 4. Say so.
- `fanout.py` is pure stdlib; only the arXiv source shells out to the
  arxiv-search script (located dynamically). If that script moves, fix the
  locator in `fanout.py`.

---
> Source: [PlutoLei/multi-source-paper-search](https://github.com/PlutoLei/multi-source-paper-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
