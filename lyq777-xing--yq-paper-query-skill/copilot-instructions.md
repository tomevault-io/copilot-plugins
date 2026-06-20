## yq-paper-query-skill

> >-


# Paper Query Skill

## What I Do

- Search 7+ academic databases in parallel with automatic retry and fallback
- **Filter: Only show published papers** (journal/conference proceedings, exclude unpublished preprints)
- Deduplicate and merge results by DOI, arXiv ID, or title similarity
- Annotate venue tiers using CCF (A/B/C), CAS-JCR (Q1-Q4), and SCI (Q1-Q4)
- Translate paper titles from English to Chinese
- Generate 2-3 sentence bilingual summaries from abstracts
- Format results as Markdown with direct links to arXiv, DOI, Semantic Scholar
- Auto-save results to `papers/query_<timestamp>.md`

---

## Workflow

### Step 0: Check Configuration

Read `paper-config.json` from the project root. If the file is missing:

> I notice your paper query is not set up yet. Let me help you configure it. Please run `/paper-setup` to create your profile — it takes about 2 minutes.

If the file exists, extract:
- `user.name`, `user.institution`, `user.field_of_study`
- `api_keys` for rate-limited APIs
- `preferences.databases`, `preferences.min_venue_tier`, `preferences.max_results`
- `preferences.sort_by`, `preferences.year_range`, `preferences.output`

**Dynamic Year Range**: If `preferences.year_range` is `null` or not set, default to the last 3 years:
- `year_start = current_year - 3` (e.g., 2023 if today is 2026)
- `year_end = current_year`
- User can specify explicit year range in query to override this default

### Step 1: Parse Query

From the user's query string, extract:
- **Keywords**: core search terms — generate 2-3 query variants (exact phrase, broader terms, synonyms) to maximize recall
- **Authors**: if mentioned (e.g., "papers by Hinton")
- **Year range**: if user specifies (e.g., "--year 2023-2025"), use it. Otherwise, use config default (last 3 years). **CRITICAL: Always apply a year filter** to avoid overwhelming results.
- **Venue filter**: if user mentions a specific conference/journal
- **Topic mapping**: map keywords to CS subfields for venue prioritization

**Important**: The year range defaults to the last 3 years. If the user does not specify a year and the config has `null`, use `current_year - 3` to `current_year`.

### Step 2: Parallel Database Search

Query all configured databases simultaneously using `webfetch`. 
**IMPORTANT**: Request **2x** `preferences.max_results` from each source to compensate for unpublished preprints that will be filtered later.

**General Retry Strategy for all APIs**:
- If 429 (rate limited): wait 5 seconds, retry once. If still fails, try the fallback endpoint.
- If 500/502/503 (server error): wait 2 seconds, retry once. If still fails, skip that source and note it.
- If timeout (>15s): skip and try fallback.
- If 403 (forbidden): skip immediately — this source requires authentication.

**Data Coverage Note**: Semantic Scholar and OpenAlex are meta-search engines that aggregate papers from hundreds of publishers including IEEE Xplore, ACM Digital Library, Springer, Elsevier, and others. Even when IEEE/ACM direct APIs fail, their papers ARE still represented in the results — you just get them through S2/OpenAlex instead of directly. Do NOT treat IEEE/ACM direct API failure as missing data.

#### 2.1 arXiv API (with rate-limiting discipline)

arXiv enforces strict rate limits: **no more than 1 request per 3 seconds**. Violations result in 429 + temporary ban.

**Query strategy** — use a single targeted request:
```
URL: http://export.arxiv.org/api/query?search_query=all:{encoded_keywords}&start=0&max_results={2x_max_results}&sortBy=relevance
```
Parse XML response. Extract: id, title, summary, authors (list), published (date), doi, journal_ref, primary_category, link.

**Rate limit handling**:
- If 429: wait 20 seconds (arXiv ban cooldown), retry once. If still fails, skip — S2 and OpenAlex already cover arXiv papers.
- Do NOT send multiple rapid requests to arXiv. One query per search is sufficient.

**Published-only check**: After parsing, flag papers that have a `journal_ref` or `doi` as PUBLISHED. Papers with neither are preprints — keep them for dedup but mark as unpublished candidate.

#### 2.2 Semantic Scholar API (with rate-limit handling)

**Primary endpoint** — try this first:
```
URL: https://api.semanticscholar.org/graph/v1/paper/search?query={encoded_keywords}&limit={2x_max_results}&year={year_start}-{year_end}&fieldsOfStudy={topic}&fields=title,year,authors,externalIds,url,abstract,venue,citationCount,publicationTypes,openAccessPdf,publicationDate,journal
```
If API key configured, add: `x-api-key: {key}`

**Fallback endpoint** — use if primary returns 429:
```
URL: https://api.semanticscholar.org/graph/v1/paper/search/bulk?query={encoded_keywords}&year={year_start}-{year_end}&fields=title,year,authors,externalIds,url,abstract,venue,citationCount,publicationTypes,journal
```

**No-key workaround** — if both endpoints rate-limit and no API key is configured:
- Inform user: "Semantic Scholar API 请求被限。建议在 https://www.semanticscholar.org/product/api 免费注册 API key 以提高限额。"
- Skip Semantic Scholar, proceed with other databases.

Parse JSON response (`data` array). Extract: title, year, authors, externalIds (DOI, ArXiv), url, abstract, venue, citationCount, publicationTypes, journal.

#### 2.3 DBLP API (with fallback)

**Primary endpoint**:
```
URL: https://dblp.org/search/publ/api?q={encoded_keywords}&h={2x_max_results}&format=json
```

**Fallback endpoint** — use author-title search if primary fails:
```
URL: https://dblp.org/search/publ/api?q={encoded_keywords}&h={2x_max_results}&format=json&c=0
```

**Second fallback** — use simpler search with f=0:
```
URL: https://dblp.org/search/publ/api?q={encoded_keywords}&h={2x_max_results}&f=0&format=json
```

If DBLP returns 500 after all retries, skip it — DBLP has known stability issues. Note in results.

Parse JSON. Extract from `result.hits.hit[]`: title, year, venue (from `info.venue`), authors (from `info.authors.author[]`), doi (from `info.doi` or `info.ee`).

#### 2.4 PubMed E-utilities
```
Step 1 (search): https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term={encoded_keywords}&retmax={2x_max_results}&retmode=json&mindate={year_start}&maxdate={year_end}&datetype=pdat
Step 2 (fetch): https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&id={comma_separated_ids}&retmode=json
```
Parse JSON. Extract: title, authors, pubdate, source (journal), doi, articleids.

#### 2.5 IEEE Xplore (API key required)

IEEE Xplore has no public REST API. The search page aggressively blocks automated access (403). There are two paths:

**Path A — With API key** (configured in `paper-config.json.api_keys.ieee`):
```
URL: https://ieeexplore.ieee.org/rest/search
Method: POST
Headers:
  Content-Type: application/json
  Accept: application/json
  apikey: {ieee_api_key}
Body: {"queryText": "{keywords}", "returnType": "SEARCH", "highlight": false, "rowsPerPage": {2x_max_results}, "ranges": ["{year_start}_{year_end}_Year"]}
```

**Path B — Without API key** (most users):
Skip IEEE Xplore direct search entirely. The IEEE papers ARE already covered by Semantic Scholar and OpenAlex — they index IEEE Xplore content including all conference proceedings and journals. Your results will include IEEE papers via these sources.

**How to get an IEEE API key**:
1. Visit https://developer.ieee.org/
2. Register for a free account (requires institutional email or IEEE membership)
3. Request API access — approval may take days
4. Add the key to `paper-config.json`: `"api_keys": { "ieee": "your-key-here" }`

**If 403 or auth error**: Skip and note "IEEE Xplore 需 API key 授权。IEEE 论文已通过 Semantic Scholar/OpenAlex 覆盖。"

#### 2.6 ACM Digital Library (limited access)

ACM Digital Library blocks automated web scraping (403). Direct search is unreliable.

**With institutional access** (if user can authenticate):
Try a single webfetch attempt:
```
URL: https://dl.acm.org/action/doSearch?AllField={encoded_keywords}&pageSize={2x_max_results}&startPage=0&AfterYear={year_start}&BeforeYear={year_end}
```
Parse HTML for paper entries if response is successful.

**Without institutional access** (most users):
Skip ACM DL direct search entirely. ACM papers (SIGMOD, SIGCOMM, CHI, CCS, OSDI, etc.) ARE indexed by DBLP, Semantic Scholar, and OpenAlex — all three cover ACM venues comprehensively. No data is lost by skipping ACM direct.

**If 403 or blocked**: Skip silently. Note "ACM Digital Library 拒绝自动访问 — ACM 论文已通过 DBLP/Semantic Scholar/OpenAlex 覆盖。"

#### 2.7 Google Scholar (webfetch — last resort fallback)
Only use if fewer than 5 results returned from all other databases combined.
```
URL: https://scholar.google.com/scholar?q={encoded_keywords}&as_ylo={year_start}&as_yhi={year_end}&num={max_results}
```
Parse HTML. Extract title, authors, year, venue, cited-by count.

**Warning**: Google Scholar frequently blocks automated access. If blocked or returns empty, skip silently — do not treat as error.

#### 2.8 Additional Open Access Sources (if results are still insufficient)

**OpenAlex API** (free, no key needed):
```
URL: https://api.openalex.org/works?search={encoded_keywords}&filter=publication_year:{year_start}-{year_end},type:article|proceedings&per_page={2x_max_results}&sort=publication_date:desc
```
This is an excellent fallback — OpenAlex has broad coverage and is fully open access.

**CORE API** (free, no key needed):
```
URL: https://api.core.ac.uk/v3/search/works?q={encoded_keywords}&yearFrom={year_start}&yearTo={year_end}&limit={2x_max_results}
```

### Step 2.5: Published-Paper Filter (CRITICAL)

After collecting raw results from all databases, apply this filter to keep ONLY published papers:

A paper is considered PUBLISHED if it has ANY of the following:
- **DOI** assigned (from any source)
- **Journal reference** present (arXiv `journal_ref` field)
- **Venue + year** clearly specified (conference name + year, or journal name + volume/issue)
- **In DBLP** (DBLP only indexes published CS papers)
- **In PubMed** with PMID (PubMed only indexes published papers)
- **publicationTypes** from Semantic Scholar indicates "JournalArticle" or "Review"

A paper is UNPUBLISHED (exclude) if:
- Only has arXiv ID but NO DOI, NO journal_ref, NO venue
- Only appears on Google Scholar without verifiable venue

**Filtering logic**: Remove all unpublished papers from the candidate list. Keep a count of how many were removed for reporting.

**Exception**: If after filtering, fewer than 3 papers remain, fall back to the broader candidate list and mark the report header: "⚠️ 大部分结果为预印本/未发表论文，仅有 {count} 篇已发表。"

### Step 3: Deduplicate and Merge

After collecting results from all databases:

1. **Normalize titles**: lowercase, remove punctuation, strip extra whitespace
2. **Match by DOI**: exact DOI match → merge
3. **Match by arXiv ID**: exact arXiv ID match → merge
4. **Match by title similarity**: if title edit distance < 10% of length → likely same paper
5. **Merge data**: for matched papers, combine metadata from all sources, preferring:
   - DOI from Semantic Scholar or CrossRef
   - Citation count from Semantic Scholar
   - Abstract from arXiv (usually most complete)
   - Venue info from DBLP (usually most accurate)

### Step 4: Venue Tier Annotation

For each paper, check its venue (conference or journal) against the tier database below. Assign ALL applicable tiers.

**Tier Badge Format**: `🏆CCF-A ⭐SCI-Q1 📊CAS-Q1`

#### Tier Database

##### AI / Machine Learning
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| NeurIPS | Conference | A | — | — |
| ICML | Conference | A | — | — |
| AAAI | Conference | A | — | — |
| IJCAI | Conference | A | — | — |
| ICLR | Conference | A* | — | — |
| UAI | Conference | B | — | — |
| AISTATS | Conference | B | — | — |
| ECAI | Conference | B | — | — |
| ACML | Conference | C | — | — |
| TPAMI | Journal | A | Q1 | Q1 |
| JMLR | Journal | A | Q1 | Q1 |
| AIJ | Journal | A | Q1 | Q1 |
| Neural Networks | Journal | B | Q1 | Q1 |
| MLJ | Journal | B | Q2 | Q2 |
| Neurocomputing | Journal | C | Q1 | Q2 |
| IJON | Journal | C | Q1 | Q2 |
| Pattern Recognition | Journal | B | Q1 | Q1 |

##### Computer Vision
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| CVPR | Conference | A | — | — |
| ICCV | Conference | A | — | — |
| ECCV | Conference | B | — | — |
| BMVC | Conference | C | — | — |
| ACCV | Conference | C | — | — |
| IJCV | Journal | A | Q1 | Q1 |
| TIP | Journal | A | Q1 | Q1 |
| CVIU | Journal | B | Q2 | Q2 |
| PR Letters | Journal | C | Q3 | Q3 |
| IET-CV | Journal | C | Q4 | Q4 |

##### Natural Language Processing
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| ACL | Conference | A | — | — |
| EMNLP | Conference | A | — | — |
| NAACL | Conference | B | — | — |
| COLING | Conference | B | — | — |
| CoNLL | Conference | B | — | — |
| EACL | Conference | C | — | — |
| CL | Journal | A | Q1 | Q1 |
| TACL | Journal | A | Q1 | Q1 |
| NLP Engineering | Journal | B | Q2 | Q2 |
| ACM-TALLIP | Journal | C | Q4 | Q4 |

##### Computer Systems / Architecture
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| OSDI | Conference | A | — | — |
| SOSP | Conference | A | — | — |
| ASPLOS | Conference | A | — | — |
| EuroSys | Conference | A | — | — |
| USENIX ATC | Conference | A | — | — |
| FAST | Conference | A | — | — |
| ISCA | Conference | A | — | — |
| MICRO | Conference | A | — | — |
| HPCA | Conference | A | — | — |
| SC | Conference | A | — | — |
| PPoPP | Conference | A | — | — |
| PODC | Conference | B | — | — |
| ICDCS | Conference | B | — | — |
| Middleware | Conference | B | — | — |
| TOCS | Journal | A | Q1 | Q1 |
| TPDS | Journal | A | Q1 | Q1 |
| TC | Journal | A | Q1 | Q1 |
| TACO | Journal | A | Q1 | Q1 |
| JPDC | Journal | B | Q2 | Q2 |
| FGCS | Journal | C | Q1 | Q2 |

##### Computer Networks
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| SIGCOMM | Conference | A | — | — |
| NSDI | Conference | A | — | — |
| MobiCom | Conference | A | — | — |
| INFOCOM | Conference | A | — | — |
| SenSys | Conference | B | — | — |
| ICNP | Conference | B | — | — |
| CoNEXT | Conference | B | — | — |
| MobiSys | Conference | B | — | — |
| TON | Journal | A | Q1 | Q1 |
| JSAC | Journal | A | Q1 | Q1 |
| TMC | Journal | A | Q1 | Q1 |
| TWC | Journal | B | Q1 | Q1 |
| ComNet | Journal | B | Q2 | Q2 |

##### Databases
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| SIGMOD | Conference | A | — | — |
| VLDB | Conference | A | — | — |
| ICDE | Conference | A | — | — |
| PODS | Conference | B | — | — |
| CIKM | Conference | B | — | — |
| EDBT | Conference | B | — | — |
| DASFAA | Conference | B | — | — |
| TKDE | Journal | A | Q1 | Q1 |
| VLDBJ | Journal | A | Q1 | Q1 |
| ISJ | Journal | B | Q1 | Q1 |
| DKE | Journal | B | Q2 | Q2 |
| WWWJ | Journal | B | Q3 | Q3 |

##### Security / Privacy
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| IEEE S&P | Conference | A | — | — |
| ACM CCS | Conference | A | — | — |
| USENIX Security | Conference | A | — | — |
| NDSS | Conference | A | — | — |
| ESORICS | Conference | B | — | — |
| RAID | Conference | B | — | — |
| ACSAC | Conference | B | — | — |
| ASIACRYPT | Conference | B | — | — |
| TDSC | Journal | A | Q1 | Q1 |
| TIFS | Journal | A | Q1 | Q1 |
| Computers & Security | Journal | B | Q1 | Q2 |
| JCS | Journal | B | Q2 | Q2 |

##### Software Engineering
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| ICSE | Conference | A | — | — |
| FSE/ESEC | Conference | A | — | — |
| ASE | Conference | A | — | — |
| ISSTA | Conference | A | — | — |
| OOPSLA/SPLASH | Conference | A | — | — |
| ICSME | Conference | B | — | — |
| SANER | Conference | B | — | — |
| MSR | Conference | B | — | — |
| TSE | Journal | A | Q1 | Q1 |
| TOSEM | Journal | A | Q1 | Q1 |
| EMSE | Journal | B | Q1 | Q1 |
| JSS | Journal | B | Q1 | Q1 |
| IST | Journal | B | Q1 | Q1 |
| SPE | Journal | B | Q2 | Q3 |
| SQJ | Journal | C | Q3 | Q3 |

##### HCI / CSCW
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| CHI | Conference | A | — | — |
| UIST | Conference | A | — | — |
| CSCW | Conference | A | — | — |
| UBICOMP | Conference | A | — | — |
| MobileHCI | Conference | B | — | — |
| DIS | Conference | B | — | — |
| IUI | Conference | B | — | — |
| TOCHI | Journal | A | Q1 | Q1 |
| IJHCS | Journal | A | Q1 | Q1 |
| IWC | Journal | B | Q2 | Q2 |
| BIT | Journal | B | Q2 | Q2 |

##### Theory / Algorithms
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| STOC | Conference | A | — | — |
| FOCS | Conference | A | — | — |
| SODA | Conference | A | — | — |
| ICALP | Conference | B | — | — |
| COLT | Conference | B | — | — |
| SPAA | Conference | B | — | — |
| ISAAC | Conference | B | — | — |
| CCC | Conference | B | — | — |
| JACM | Journal | A | Q1 | Q1 |
| SICOMP | Journal | A | Q1 | Q1 |
| Algorithmica | Journal | B | Q2 | Q2 |
| TALG | Journal | A | Q1 | Q1 |
| JCSS | Journal | B | Q2 | Q2 |

##### Interdisciplinary / Bioinformatics
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| RECOMB | Conference | B | — | — |
| ISMB | Conference | B | — | — |
| MICCAI | Conference | B | — | — |
| Bioinformatics | Journal | B | Q1 | Q1 |
| PLOS-CB | Journal | B | Q1 | Q1 |
| BMC Bioinformatics | Journal | C | Q2 | Q2 |
| JBI | Journal | B | Q1 | Q1 |

##### General Top-Tier Science
| Venue | Type | CCF | SCI | CAS-JCR |
|-------|------|-----|-----|---------|
| Nature | Journal | — | Q1 | Q1 |
| Science | Journal | — | Q1 | Q1 |
| PNAS | Journal | — | Q1 | Q1 |
| Nature Communications | Journal | — | Q1 | Q1 |
| Science Advances | Journal | — | Q1 | Q1 |

#### Filtering Rule
Apply `preferences.min_venue_tier`:
- If a paper's venue is not found in any tier database, do NOT filter it out (it may be from a newer/niche venue)
- If found and tier is below configured minimum, skip the paper
- If `min_venue_tier.ccf` is empty/null, skip CCF filtering
- Apply same logic for `cas_jcr` and `sci`

### Step 5: Content Enhancement

For each paper in the final list:

#### 5.1 Title Translation
If `user.auto_translate_title` is true, translate the English title to Chinese using the LLM's native translation capability. Format:
```
{English Title}
{中文译名}
```

#### 5.2 Bilingual Summary
If `user.auto_summarize` is true and abstract is available:
1. Read the full abstract
2. Generate a concise 2-3 sentence summary in English
3. Generate a corresponding 2-3 sentence summary in Chinese
4. Keep summaries focused on: problem, method, key finding

### Step 6: Format Output

#### 6.1 Output Template

```markdown
# 论文查询结果: "{query}"
**查询时间**: {timestamp} | **用户**: {user.name} | **数据库**: {databases_list} | **结果数**: {count}

## 概览
- 检索关键词: {keywords}
- 年份范围: {year_start}–{year_end}
- 最低等级: CCF-{min_ccf} / SCI-{min_sci} / CAS-{min_cas}
- 排序方式: {sort_by}

| # | 标题 | 作者 | 年份 | 会议/期刊 | 等级 | 引用 | 链接 |
|---|------|------|------|----------|------|------|------|
{rows}

---

## 详细信息

### #1 {English Title}
**中文标题**: {Chinese Title}
**作者**: {authors}
**出处**: {venue} ({year})
**等级**: 🏆 CCF-{tier} ⭐ SCI-{q} 📊 CAS-{q}
**引用**: {citations}
**链接**: [arXiv:{id}]({arxiv_url}) | [DOI]({doi_url}) | [Semantic Scholar]({s2_url})
**概述 (EN)**: {english_summary}
**概述 (CN)**: {chinese_summary}

---

(Generated by paper-query skill)
```

#### 6.2 Link Construction
- **arXiv**: `https://arxiv.org/abs/{arxiv_id}`
- **DOI**: `https://doi.org/{doi}`
- **Semantic Scholar**: `https://api.semanticscholar.org/{paper_id}` (or use url field from API)
- **If no direct links available**: include search link `https://scholar.google.com/scholar?q={encoded_title}`

#### 6.3 Save File
If `preferences.output.save_to_file` is true:
- Filename: `papers/query_{YYYYMMDD}_{HHmm}.md`
- Use the `papers/` directory in project root

---

## Error Handling

### API Errors — Specific Recovery

| Database | Error | Recovery |
|----------|-------|----------|
| **Semantic Scholar** | 429 Rate Limit | Wait 3s → retry with fallback endpoint. If still fails and no API key: inform user about free API key, skip S2, continue |
| **Semantic Scholar** | 500/502/503 | Wait 2s → retry once. Skip if persistent |
| **DBLP** | 500 Server Error | Try fallback URL with `&f=0`. If still fails, skip. DBLP has known stability issues — note in results: "DBLP 服务暂时不可用，已跳过" |
| **DBLP** | Timeout | Skip immediately, note in results |
| **arXiv** | 429 Rate Limit | Wait 20s (arXiv ban cooldown) → retry once. If still fails, skip — S2 and OpenAlex already cover arXiv papers |
| **arXiv** | Any other error | Retry once after 3s. Skip if persistent |
| **PubMed** | 429 | Wait 1s, retry. PubMed is generally stable |
| **IEEE Xplore** | 403 / Auth required / POST blocked | Skip entirely. IEEE papers ARE covered by Semantic Scholar and OpenAlex. Note: "IEEE Xplore 需 API key — IEEE 论文已通过 S2/OpenAlex 覆盖" |
| **ACM DL** | 403 / Access denied | Skip entirely. ACM papers ARE covered by DBLP, Semantic Scholar, and OpenAlex. Note: "ACM 论文已通过 DBLP/S2/OpenAlex 覆盖" |
| **Google Scholar** | Blocked/Empty | Skip silently — expected behavior for automated access |
| **OpenAlex** | Any error | Skip — this is a bonus source |

### General Recovery Rules
- **API timeout/error after retries**: Skip that database, continue with others. Note which failed at the bottom of the report.
- **All databases failed**: Report the situation clearly, suggest the user retry later or broaden keywords.
- **0 published results found**: Report "未找到符合条件的已发表论文" and suggest:
  - Broaden keywords or use fewer/alternate terms
  - Remove year filter or expand year range
  - Lower venue tier filter
  - Note that very new papers may not yet have DOI/journal assignment
- **Very few results (<5)**: Fall back to Google Scholar and OpenAlex automatically.
- **Rate limited across multiple databases**: Explain that API keys improve reliability (especially Semantic Scholar free key).

---

## Important Notes

- Always read `paper-config.json` first before any query
- **Default year range is last 3 years** (current_year - 3 to current_year). Always apply a year filter.
- **Only show published papers**: Filter out preprints without DOI, journal_ref, or confirmed venue
- **Request 2x the target count** from APIs to compensate for filtering losses
- **Meta-search coverage**: Semantic Scholar and OpenAlex already index IEEE, ACM, Springer, Elsevier, and most other publishers. When IEEE/ACM direct APIs fail (403/blocked), their papers ARE still in your results via S2/OpenAlex. This is expected and NOT a problem.
- Use `webfetch` for all API calls and web scraping
- Process databases in parallel when possible (multiple webfetch calls)
- **Always try fallback endpoints** before giving up on a database
- If a paper appears in multiple databases, merge their metadata
- Include ALL available links for each paper (arXiv, DOI, Semantic Scholar)
- The Chinese translation should be professional/academic in tone, not too colloquial
- When venue tier is unknown, mark as "—" rather than omitting the field
- **arXiv rate limit**: minimum 3s between requests. If 429, wait 20s cooldown
- **IEEE/ACM auth**: These need API keys or institutional access. Without them, papers still appear via S2/OpenAlex
- Report which databases succeeded/failed at the bottom of the output, with coverage notes
- If DBLP returns 500, it's a known intermittent issue — do not retry more than once

---
> Source: [lyq777-Xing/yq_paper_query_skill](https://github.com/lyq777-Xing/yq_paper_query_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
