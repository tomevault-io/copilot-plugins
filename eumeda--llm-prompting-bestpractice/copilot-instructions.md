## llm-prompting-bestpractice

> 外部 LLM サービス (Anthropic Claude, OpenAI GPT, Edison Scientific Kosmos, Elicit, Perplexity 等) に投入するプロンプトを 3 ベンダー統合 best practice で設計・最適化する。XML タグ構造化・5 コンポーネント (role/task/context/format/constraints)・outcome-first 指示・Kosmos 用 single-objective 設計・Anti-pattern 検査を整合的に適用する。Triggers on: プロンプト最適化, prompt optimization, prompt engineering, Kosmos プロンプト, Edison プロンプト, GPT プロンプト, Claude プロンプト, XML タグ, prompt 設計, prompt review, 外部 LLM 投入, /LLM-prompting-bestpractice


# Skill: `/LLM-prompting-bestpractice` — 3 ベンダー統合プロンプト設計

## なぜこの skill が必要か

外部 LLM サービスに投入するプロンプト (特に Kosmos のように 12 hr 自律実行型) は、**実行中に修正できない** ため、投入前に 3 ベンダー (Anthropic / OpenAI / Edison Scientific) の公式 best practice を統合的に適用しておく必要がある。本 skill は次の 3 ステップで設計・レビューを支援する:

1. **構造化** (5 コンポーネント + XML タグ + 推奨順序)
2. **Vendor-specific 適合化** (各サービスの公式推奨と Anti-pattern)
3. **後処理 readability** (Kosmos 出力をパースする側の負荷を下げる SECTION 設計)

## When to invoke

以下のいずれかに該当する場合に invoke する:

- 外部 LLM サービス (Kosmos, Elicit, Perplexity, ChatGPT, Claude.ai 等) に投入する prompt を新規作成・レビューする
- 既存の prompt を multi-vendor で動かす (移植) ために最適化する
- ユーザーが `/LLM-prompting-bestpractice` と直接呼ぶ
- 「プロンプト最適化」「XML タグ整理」「Kosmos に投げる文章を整えて」等

- **別マシン / 自律実行用の Claude Code prompt** を作る (例: LAB-PC で長時間データ解析を 1 セッション完走させる prompt)

逆に、いま対話している Claude Code セッションへの**その場の通常 prompt** には不要。
本 skill は **(a) 外部 LLM 投入用 prompt** と **(b) 自律/別マシン実行用の agentic prompt** に適用する (後者は Template D の anti-hallucination ガードが必須)。

---

## Section 1 — 3 ベンダー共通の核原則 (6 つ)

### 原則 1: 5 コンポーネント構造 (role / task / context / format / constraints)

Anthropic Claude 4.7 は **この 5 要素パターンで訓練されており**、構造化プロンプトの方がプレーンテキストより測定可能に高品質な出力を出す ([Anthropic 2026](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices))。OpenAI GPT-5.5 も outcome-first 指示で同等の効果。

**推奨順序**:
```
1. <role>          — モデルの専門家 persona と視点
2. <context>       — フィールド固有の前提・データ来歴・なぜ重要か
3. <task>          — 単一の主目的 (research objective)
4. <instructions>  — 番号付き手順 (sequential steps)
5. <output_format> — SECTION 構造 + 制約
```

**理由**: context を task より先に置くと、Claude は task をどの枠組みで解釈すべきかを先に理解する。これは long-context (20k+ token) で 30% 精度改善 ([Anthropic Long context tips](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices))。

### 原則 2: XML タグでセクション分離

3 ベンダー共通で XML タグは parsing を unambiguous にする。Markdown header (`## `) より強い。

**標準タグ語彙 (Anthropic 公式)**:
- `<role>`, `<context>`, `<task>`, `<instructions>`, `<output_format>` — 主構造
- `<example>`, `<examples>` — 多ショット用 (3-5 件で best)
- `<documents>`, `<document index="n">`, `<document_content>`, `<source>` — 複数ファイル投入時
- `<constraint>`, `<scope_boundary>`, `<do_not>` — 禁則事項
- `<thinking>`, `<answer>` — chain-of-thought 構造化

**ネスト**: 階層がある場合は必ずネスト。例:
```xml
<documents>
  <document index="1">
    <source>dataset.csv</source>
    <document_content>...</document_content>
  </document>
</documents>
```

**注意**: タグ名は識別性が大事 (canonical name はない)。データ内容と意味が一致する名前を選ぶ。

### 原則 3: Outcome-first (path ではなく target を指定)

GPT-5.5 (2026-04) と Claude Opus 4.7 (2026-05) はいずれも **「to-do list」より「success criteria」を好む** ([OpenAI GPT-5.5 guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide))。

**Bad (path-prescriptive)**:
```
Step 1: Read the PDF.
Step 2: Extract all species names.
Step 3: Search PubMed for each species.
Step 4: Compile results into a table.
```

**Good (outcome-first)**:
```
Produce an extension table containing records not present in the canonical
database. Each new record must include source DOI, observation year, location,
and a reliability rank A/B/C. Success criterion: no record duplicates an
existing canonical entry (or a known synonym of one).
```

**理由**: モデルの reasoning engine は path を自分で最適化する。過剰指示は探索空間を狭める。

### 原則 4: 絶対ルールは sparingly に (true invariants のみ)

OpenAI GPT-5.5 (2026-04) の重要な方針転換: **「MUST/NEVER は乱発しない」**。Claude 4.7 も literal interpretation で同様の挙動。

**MUST/NEVER を使ってよいケース** (true invariants):
- safety / 倫理規定
- 必須出力フィールド
- 投入禁止データ (例: 未発表 simulation 結果)
- スコープ境界 (out-of-scope の明示)

**MUST を使ってはいけないケース** (= soft preference):
- スタイル指示 (例: "MUST use academic tone" → 単に "Write in academic tone")
- 望ましい構造 (例: "MUST use bullet lists" → "Prefer bullet lists")
- 推奨される手順 (例: "MUST search PubMed first" → "Start by searching PubMed")

濫用は逆効果 (モデルが本物の constraint を見分けられなくなる)。

### 原則 5: 否定より肯定指示

3 ベンダー共通: **「X しないで」より「Y して」**。

**Bad**: "Do not use markdown."
**Good**: "Respond in flowing prose paragraphs."

**Bad**: "Don't be vague."
**Good**: "State the species name, location, year, and reliability rank for each record."

例外: out-of-scope 明示 (これは肯定的「禁止」として有効)。

### 原則 6: canonical examples を必ず同梱 (multishot) — 最重要

Anthropic 公式が**最も信頼できる steering 手段**と明言: 「Examples are one of the most reliable ways to steer Claude's output」「3–5 個推奨」「examples are the 'pictures' worth a thousand words」([Anthropic 2026](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices); [context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents))。ルールを並べるより**例を示す**方が format・tone・判定を確実に伝える。

- **3–5 個の canonical example** を `<example>` (複数なら `<examples>`) で囲んで同梱する。
- 例は **relevant** (実 use case を写す)・**diverse** (edge case を含み、意図しないパターンを拾わせない)・**structured** (タグで指示と分離)。
- 出力フォーマット・判定基準・report 冒頭文・table の 1 行 等、「最終成果物が何であるべきか」の見本を最低 1 つ示す。
- **例が無いプロンプトは原則 incomplete** とみなす (Section 4 checklist の必須項目)。

---

## Section 2 — Vendor-specific nuances

### 2.1 Anthropic Claude (Opus 4.8 / 4.7 / 4.6, Sonnet 4.6)

**Strong points**:
- XML タグの認識が最も堅牢 (3 ベンダー中 best)
- Long-context (200k+) で query を末尾配置すると 30% 精度改善
- `<thinking>...</thinking>` で chain-of-thought を明示可能
- multishot (3-5 例を `<examples>` で囲む) が信頼性高い

**Claude Opus 4.8 / 4.7 特有 (2026)**:
- **literal instruction following** (4.8 で更に強化): 「全セクションに適用」と書かないと最初のセクションだけ処理される。一般化させたい指示は scope を明示
- subagent spawning が default で減少 → 必要なら明示的に指示
- effort パラメータが strict に効く → 複雑/agentic タスクは `high`〜`xhigh` 推奨
- **response length を task 複雑度で自動調整** (4.8): 一定の verbosity が要るなら明示
- positive examples の方が "don't do X" より効く。**agentic coding では investigate-before-answering / overeagerness 抑制 / temp file cleanup を明示** (Template D)

**Anti-pattern**:
- prefilled response (4.6 以降サポート停止)
- 過剰な markdown bold/italics (出力にも反映されてしまう)
- vague terms ("important", "high quality") → 具体基準で書く

### 2.2 OpenAI GPT (GPT-5, 5.1, 5.2, 5.5)

**Strong points**:
- structured output (JSON schema) が API レベルで強制可能
- tool use (function calling) が安定
- reasoning effort 設定 (default: medium)

**GPT-5.5 特有 (2026-04 update)**:
- "fresh baseline" 推奨 (古いプロンプトを引きずらない)
- **outcome-first prompts** (success criteria + constraints + context のみ、path 指定不要)
- absolute rules は invariants のみ
- stopping rules を明示 (long-running / tool-heavy task)

**Anti-pattern**:
- 古い "Let's think step by step" 系の手順命令 (5.5 では逆効果)
- 過剰な absolute rules ("MUST", "NEVER") の濫用

### 2.3 Edison Scientific Kosmos (12 hr autonomous research agent)

**核原則** ([Edison 公式 best practices](https://edisonscientific.com/articles/kosmos-best-practices-guidelines)):

1. **Single, well-defined research objective** (複数並列指示は精度を下げる)
2. **反復の余地**: 数本の論文や 1 回の解析で答えが出る質問は不適。「scientific collaborator が weeks-months で完成できる規模」が target
3. **Sufficient scientific context**: フィールド固有の非自明な前提を明示。「新しく加わった有能な同僚に説明するように」
4. **Intuitively labeled columns + supplementary description**: column_description.csv 必須
5. **Complex high-dimensional data 推奨**: 単純リストより多変数・多時点・多サンプル
6. **5 GB 上限・ファイル数無制限**

**Strong objectives 例 (Edison 提供)**:
- mechanistic hypothesis を含む
- experimental design + time-course parameters
- control variables を明示
- follow-up analytical strategies

**Weak objectives (避けるべき) 例**:
- "tissue with lower ECM pathway expression" (context 不足)
- "genes differentially expressed at p < 0.05" (単純フィルタ)
- "explore relationship between X and Y" (vague)

**Kosmos-specific output design**:
- SECTION A/B/C 構造で出力指定 (downstream パース容易性)
- 各 SECTION に「内容と format」を明示 (Table / Plain text / List)
- 引用は DOI 付きで要求

**Kosmos が苦手な task**:
- single-step 抽出 (= Elicit の方が向く)
- リアルタイム対話 (= ChatGPT の方が向く)
- 数値計算自体 (= Wolfram 等の方が向く)

### 2.4 比較サマリ

| 観点 | Claude | GPT-5.5 | Kosmos |
|------|--------|---------|--------|
| 主用途 | general reasoning + agentic | general + coding | autonomous research (12hr) |
| XML タグ親和性 | 最高 (訓練済) | 高 | 中-高 (記載なしだが効く) |
| Path vs outcome | outcome-first 推奨 | outcome-first 強推奨 | outcome-first 必須 |
| Absolute rules | sparingly | sparingly | 投入禁止データ等は明示 |
| 多ファイル | `<documents>` ネスト | structured outputs | files 添付 (5GB 上限) |
| 単一目的 | 推奨 | 推奨 | **必須** |

---

## Section 3 — XML テンプレート (3 用途)

### Template A — 一般的な Claude / GPT 投入 (single-task)

```xml
<role>
You are a [domain expert] specializing in [subdomain]. Your audience is
[user role] with knowledge level [level].
</role>

<context>
[Field-specific premises, data provenance, why this matters]
</context>

<task>
[Single research/task objective in 1-2 sentences]
</task>

<instructions>
1. [Step 1, sequential]
2. [Step 2]
3. [Step 3]
</instructions>

<output_format>
Provide your output in the following structure:
- SECTION A — [name]: [content type, ~length]
- SECTION B — [name]: [content type, ~length]
</output_format>

<scope_boundary>
The following are explicitly OUT OF SCOPE:
- [Out-of-scope item 1]
- [Out-of-scope item 2]
</scope_boundary>
```

### Template B — Kosmos 投入 (12hr autonomous research)

```xml
RESEARCH OBJECTIVE
[Single, outcome-first sentence describing what discovery the agent should
make. The answer must NOT be obvious from a few papers.]

RESEARCH CONTEXT
[Field-specific premises framed for "a capable colleague who just joined
the team": what's already known, what's not, what's atypical about your
group's data/method, downstream use of the output]

CANONICAL DATA
[List attached files with one-line description each:
 - dataset.csv (N records, columns described in column_description.csv)
 - Author_etal_YYYY.pdf (supplementary primary source)
 - ...]

INSTRUCTIONS

1. [Sequential instruction 1, outcome-first]
   Sub-criterion: [success criterion]
2. [Instruction 2]
   Sub-criterion: [success criterion]
...
N. Output structure
   SECTION A - [name]: [Table | Prose ≤ X words | List]
   SECTION B - [name]: [Table | Prose ≤ X words | List]
   SECTION C - [name]: ...

SCOPE BOUNDARY
The following are explicitly OUT OF SCOPE:
- [Out-of-scope item 1]
- [Out-of-scope item 2]

REQUIRED CITATION FORMAT
All claims and new records must cite the primary source with DOI.
```

**Note**: Kosmos は Anthropic 公式 XML タグを必須としないが、SECTION 構造の明示は output parse のため必須。Single-objective 規則を死守。

### Template C — 多ファイル long-context 投入

```xml
<documents>
  <document index="1">
    <source>file1.pdf</source>
    <type>primary_target</type>
    <document_content>{{file1_text}}</document_content>
  </document>
  <document index="2">
    <source>file2.csv</source>
    <type>canonical_data</type>
    <document_content>{{file2_text}}</document_content>
  </document>
</documents>

<task>
[Task description, AFTER the documents]
</task>

<instructions>
1. First, quote the relevant passages from each document in <quotes> tags.
2. Then, perform the analysis based on those quotes.
</instructions>
```

**理由**: documents を先頭に置き、task と instructions を末尾に置くと long-context で 30% 精度改善 (Anthropic 2026)。

### Template D — agentic / autonomous Claude Code prompt (別マシン・長時間自律実行)

別マシンで Claude Code を自律実行させる prompt (例: ローカルデータ解析を 1 セッションで完走) には、Template A の構造に加えて **anti-hallucination ガードと agentic 規律**を必ず入れる。これらが無いと、未読ファイルの内容を推測したり、存在しないデータで結論を書いたり、過剰実装に走る。

```xml
<role>…</role>
<context>…(データの所在・前提・なぜ重要か)…</context>
<task>…(単一目的)…</task>

<investigate_before_answering>
Never speculate about code or data you have not opened. If a file is referenced,
you MUST read it before using its contents. Make claims and report numbers ONLY
from files you have actually read. If a required file is not found, report the gap
explicitly — never fabricate data, paths, or results.
</investigate_before_answering>

<instructions>
1. … (番号付き手順 + 各 step に success criterion)
</instructions>

<examples>
  <example>…(成果物の 1 行見本: data_inventory の行 / 判定の 1 例 / 報告冒頭の結論文)…</example>
</examples>

<output_format>…(成果物・SECTION 構造)…</output_format>

<scope_boundary>
Out of scope:
- 直接要求されていない feature 追加・refactor・"改善" (overeagerness 抑制)。
- 結論を変えない大規模な追加実行 (最小の gap 埋めのみ)。
- データが支持しない主張。不確実性は limitation として正直に書く。
</scope_boundary>

<housekeeping>
- 反復用の一時 script / 中間ファイルを作ったら、最後に削除する。
- 高コストな処理 (長時間 run 等) は勝手に起動せず、必要なら列挙して確認を仰ぐ。
- テストを通すための hard-code や一回限りの workaround をしない。一般解を書く。
</housekeeping>
```

**根拠**: Anthropic 公式の agentic 指針 — "investigate before answering" (hallucination 最小化)、"avoid overeagerness / keep changes minimal"、"reduce file creation: clean up temp files"、"avoid hard-coding / passing tests"、Opus 4.8 の literal following (scope を明示せよ) ([Anthropic prompting best practices, agentic systems](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices))。

---

## Section 4 — Self-review checklist (投入前 14 項目)

投入前に以下を機械的にチェック:

```
[ ] 1. 5 コンポーネント (role / context / task / instructions / output_format) が揃っている
[ ] 2. XML タグまたは明確な SECTION ヘッダで分離されている
[ ] 3. Single objective (複数目的並列でない)
[ ] 4. Outcome-first (success criteria 明示、path 過指定なし)
[ ] 5. Absolute rules (MUST/NEVER) は invariants のみ
[ ] 6. 否定指示より肯定指示優先 (例外: scope boundary)
[ ] 7. context が task より先に置かれている
[ ] 8. 多ファイル投入時は <documents><document index></document></documents> 構造
[ ] 9. Long-context (20k+) では documents を先頭、task を末尾
[ ] 10. SECTION 出力構造が downstream パースに適合 (Table / Prose / List 明示)
[ ] 11. Out-of-scope items が <scope_boundary> で明示
[ ] 12. 引用は DOI 付き要求 (research task の場合)
[ ] 13. canonical examples (3-5) を <example>/<examples> で同梱 — 成果物の見本を最低 1 つ【必須・原則 6】
[ ] 14. scope を literal に明示 (Opus 4.8 は「全セクションに適用」等を書かないと最初だけ処理する)
```

**Agentic / autonomous Claude Code prompt 追加 4 項目** (Template D):
```
[ ] D1. <investigate_before_answering> がある (未読ファイルを推測しない・数値を捏造しない・無ければ gap 報告)
[ ] D2. overeagerness 抑制 (直接要求外の feature/refactor/改善をしない) が scope に明示
[ ] D3. housekeeping (一時ファイル削除・高コスト処理は確認・hard-code しない) が明示
[ ] D4. 高コスト/破壊的処理の前に確認を仰ぐ指示がある
```

**Kosmos 専用追加 6 項目**:
```
[ ] K1. 12 hr に値する研究目的か (1 hr で終わる task は Elicit 向き)
[ ] K2. Column 名が intuitive か (略語の場合は description csv 添付)
[ ] K3. データが high-quality processed か (raw sequencing 等は不可)
[ ] K4. 総ファイルサイズ < 5 GB
[ ] K5. mechanistic hypothesis を含むか (探索的すぎないか)
[ ] K6. follow-up analytical strategies が指示されているか
```

---

## Section 5 — Anti-pattern カタログ

### A1. Multi-objective parallel
**Bad**:
> Extend the database AND analyze the latitudinal gradient AND identify
> publication bias.

**Why bad**: モデルが優先順位を自律決定 → 各 task が浅くなる
**Fix**: 別 run に分割

### A2. Path over-specification
**Bad**:
> First read the PDF. Then list each species. Then for each species, search
> PubMed using exactly the query "[species name] AND review". Then ...

**Why bad**: モデルの reasoning 探索を狭める
**Fix**: outcome + success criteria を書き、path はモデルに任せる

### A3. Vague qualitative terms
**Bad**:
> Identify the most important papers. Make sure the analysis is high quality.

**Why bad**: "important" / "high quality" は計測不能
**Fix**: 具体基準 (DOI 付き、5 件以上、PGLS 解析を含む等)

### A4. MUST/NEVER spam
**Bad**:
> MUST use academic tone. MUST cite at least 5 papers. MUST format as
> markdown. NEVER use bullet points. MUST be concise. NEVER use first
> person.

**Why bad**: 本物の invariant が埋もれる、モデルが過剰防衛的になる
**Fix**: invariant のみ MUST/NEVER。preference は普通の動詞。

### A5. Confirmation bias 注入
**Bad** (Kosmos に「独立評価」を求めながら、自分の批判メモを補助投入):
> Re-evaluate paper X (a 2024 study).
> Supplementary: my_critique_of_X.md (= 既に否定的結論を持つメモ)

**Why bad**: 独立性を破壊する
**Fix**: 批判メモは渡さず、対象論文の事実だけ与える。誤読防止のための minimal な事実 (e.g., 「論文 X は A>B と主張している」) のみ context に書く

### A6. Schema-spec mismatch
**Bad**:
> Produce a table with the same schema as the input CSV.
> (input CSV は 28 列、output は 15 列を想定しているが明示していない)

**Why bad**: モデルが input 全列をコピーする → downstream パースが破綻
**Fix**: output schema を明示列挙

### A7. 例ゼロ (all rules, no examples)
**Bad**:
> (role/context/task/instructions/output を全部ルールで書き、example を 1 つも入れない)

**Why bad**: 公式が最重要 steering 手段と明言する examples を欠く。format/判定/tone がぶれる。よくある optimizer の盲点。
**Fix**: 成果物の見本を 3–5 個 `<examples>` で同梱 (原則 6)

### A8. agentic prompt に anti-hallucination ガードが無い
**Bad**:
> (ローカルデータを解析させる自律 prompt で「ファイルを探して解析して報告」とだけ書く)

**Why bad**: 未読ファイルの内容を推測し、見つからないデータで数値・結論を捏造し得る。
**Fix**: `<investigate_before_answering>` (未読を推測しない・捏造しない・無ければ gap 報告) と housekeeping を入れる (Template D)

---

## References

### Primary docs (3 vendor)
- [Anthropic — Prompting best practices (Claude Opus 4.8, 2026)](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) — clarity / examples / XML / thinking / **agentic systems** (literal following, overeagerness, hallucination 最小化, file cleanup)
- [Anthropic — Effective context engineering for AI agents (2026)](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — right altitude / minimal-but-not-short / examples over exhaustive rules
- [Anthropic — Use XML tags](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags)
- [OpenAI — GPT-5 Prompting Guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide)
- [OpenAI — GPT-5.5 Prompting Guide (2026-04)](https://simonwillison.net/2026/apr/25/gpt-5-5-prompting-guide/)
- [Edison Scientific — Kosmos best practices](https://edisonscientific.com/articles/kosmos-best-practices-guidelines)
- [Edison Scientific — Announcing Kosmos](https://edisonscientific.com/articles/announcing-kosmos)
- [Kosmos paper (arXiv 2511.02824)](https://arxiv.org/abs/2511.02824)

### Foundational research papers
- arXiv 2411.10541 — Format comparison (Markdown best for large models)
- arXiv 2408.02442 — Format restrictions reduce reasoning (EMNLP 2024)
- arXiv 2502.04295 — Content-format optimization
- arXiv 2305.13062 — HTML table 6.76% > Markdown table (WSDM 2024)
- arXiv 2406.06608 — Prompt Report (systematic survey)
- arXiv 2309.11495 — Chain-of-Verification reduces hallucination
- arXiv 2312.16171 — Principled Instructions

---
> Source: [eUmeda/LLM-prompting-bestpractice](https://github.com/eUmeda/LLM-prompting-bestpractice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
