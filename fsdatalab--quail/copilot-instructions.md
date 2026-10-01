## quail

> Write like two engineers talking at a whiteboard.

# Writing

Write like two engineers talking at a whiteboard.

- Lead with the answer, then the reasoning. Short sentences, one idea
  each.
- Use bullet points for anything longer than a few sentences.
- Use everyday words. Define an unavoidable technical term once, in
  one sentence, then use it the same way every time.
- No analogies, metaphors, or dramatic shorthand. Say "the run is on
  Modal", not "the flight is in the air"; say "saved", "finished",
  "running", not "banked", "landed", "armed", "healthy".
- Give an example with "For example," and all of its context: "For
  example, imagine a table where each row is an agent trace…".
- Give numbers, and say what each is compared with: "52.3 s, compared
  with 47 s for the unavoidable work alone".
- Do not start a sentence with a number.
- When something failed or is uncertain, say so, and say what would
  settle it.
- Call the KV cache "KV".
- Say "run" or "configuration" for the runs of an experiment, never
  "arm".

## Code

- Comments state constraints the code cannot show, nothing else.
- Docstrings follow Google style: a one-line summary, a blank line,
  then optional Args, Returns, and Raises. Say what the code does, not
  why it was written or what it replaced.
- Use one name for each concept across the planner, the executor, the
  docs, and the metrics.

# Checks

CI runs these on every pull request. Run them before pushing:

    uv run ruff check quail tests experiments tools
    uv run python tools/check_long_strings.py
    uv run vulture
    uv run pytest -q

- Ruff enforces line length 88, import order, naming, and Google style
  docstrings (`[tool.ruff]` in `pyproject.toml`).
- `tools/check_long_strings.py` flags string literals over 200
  characters. Long content is allowed when named as content (a name
  containing PROMPT, TEMPLATE, SQL, QUERY, HTML, or TEXT), kept in a
  prompts module or folder, or plainly HTML or SQL. Docstrings are
  exempt. Shorten long messages and log lines.

# Names

- The project is Quail (QUery-Aware Inference Layer); the package is
  `quail`.
- quail-b, the benchmark, lives in
  https://github.com/fsdatalab/quail-bench and is installed here as
  `quail_b`, pinned to a commit in `pyproject.toml`. `quail/bench/` is
  Quail's runner for it. Change queries or labels there first, then
  move the pin here.
- The execution mechanisms are "pipelining", "token-based admission",
  "KV rewind", and "prefix sharing". "Chain mode" is the code name for
  KV rewind; prefer "KV rewind" in prose.
- The attention paths are "unified" and "tree".
- Compare against "stock vLLM", and say which submission strategy it
  used: operator-at-a-time or pipelining.
- Every `Session` takes an `EngineConfig` that names `model` and
  `device`; neither has a default. `gpus` defaults to 1 and `backend`
  to `"quail"`.

# Scope

- Filter and join queries on Qwen3 4B fp8, Qwen3 32B fp8, or
  DiffusionGemma 26B-A4B fp8, on one H100 per model copy. No
  tensor-parallel weight sharding.
- `AI.CLASSIFY`, `AI.EXTRACT`, and `AI.MAP` are on the roadmap.
  Open-ended generation, speculation, and forking are not supported.

# What Quail builds on and learns from

Quail is built from these projects. Read their code and docs before
reinventing a piece, and hold Quail's code and docs to their
standard.

Built on:

- **vLLM**: model definitions, weight loading, and kernels; also the
  main baseline.
- **FlashAttention**: the attention kernels, over paged KV.
- **DeepGEMM**: the FP8 matrix multiplies.
- **Triton**: Quail's fused kernels (RMSNorm plus quantize, QK-norm
  plus RoPE, SiLU plus quantize, the attention merge).
- **Apache Arrow and Acero**: tables, streams, and the relational
  operators in query plans.
- **Substrait**: the plan format quail-b queries are written in.
- **sqlglot**: SQL parsing.
- **Hugging Face** (`transformers`, `huggingface_hub`, `datasets`):
  tokenizers, checkpoints, and benchmark data.
- **Modal**: every GPU run and the result volumes.
- **SGLang**: a second baseline.

Learned from. Treat these as whole projects to study, not single
features: their APIs, their code layout, their tests, their docs, and
how they release and explain changes.

- **Apache Spark**: the model for Quail as a whole.
  - Catalyst, for logical and physical plans rewritten by named rules
    (Quail's planner rules).
  - The DataFrame and SQL APIs, for Quail's builder and `sql()`.
  - Spark Connect, for a thin client talking to a remote server
    (Quail Server and the remote `Session`).
  - `EXPLAIN` and the Spark UI, for showing a plan and what it cost
    (`explain()`, `explain(analyze=True)`).
- **Apache DataFusion**: an Arrow-native query engine built to be
  extended, with clear extension points (Quail's extension registry,
  custom operators, and rules) and docs for each.
- **DuckDB**: an engine that is easy to embed and whose docs answer a
  question in one short page with a runnable example.
- **Apache Arrow**: columnar data and Arrow Flight for moving results
  between processes.
- **vLLM**: the serving engine Quail runs beside and compares
  against; its paper and docs explain one mechanism per section, such
  as paged KV and prefix caching, with a small example.
- **Modal**: docs that start with what a feature does, then a small
  example with full context.
- **Selinger et al. (1979)**: cost-based join ordering, which the
  planner follows.

# Experiments

- Every engine run goes through Modal; there is no local GPU.
- Do not create Modal app names. Attach GPU cells to
  `quail-milestone1` and the worker to `quail-engine`; caches and warm
  state belong to those apps.
- Tee every Modal run to a file; the CLI drops old log lines.
- Do not save Modal return values to local JSON. Print the function
  call id (`fc-...`) into the tee file, and fetch the result later
  with `modal.FunctionCall.from_id("<id>").get()`. An Arrow Flight
  request has no function call id; print the Flight query id and the
  result volume path instead.
- Results live on the `quail-results` volume, summaries and per-item
  records alike. Do not commit them. Cite them by volume path, such as
  `/results/ablations/<file>.json`.
- State the prediction before the run, then report the result against
  it.
- Work from measured constants first. Run one confirming cell, not a
  sweep, unless the sweep is the point.
- Configure the baseline as well as Quail. If Quail gets a setting
  from the plan, the baseline gets the equivalent one. Report the
  setting with the result.
- For every benchmark query, report query time, throughput, and GPU
  cost:
  - Throughput is input tokens per second: the full length of every
    prompt the query evaluated, summed, counting a shared prefix every
    time whether or not its KV was reused, divided by query time.
    quail-b reports it as `input_tokens_per_second`. Give document and
    pair counts beside it as plain counts, not as throughput.
  - `$/query` is query time in hours times the number of GPUs times
    `quail.specs.H100_USD_PER_HOUR` ($3.9492, from
    https://modal.com/pricing).
  - Query time and `$/query` exclude model startup. Report startup
    separately if it matters.

# Reports and plots

- Reports and plots do not go on branches that target `main`, and a
  PR does not get its own report.
- They live on the `cursor/reports-dev-f955` branch, with the plot
  code, figures, and `engine-wiki.md`. Start report work there and
  target changes back to it.
- A plot script takes the directory of files pulled from the volume
  as its first argument, lists the `modal volume get` commands in its
  docstring, and computes percentages and ratios itself.
- Put one script per report in `reports/make_<slug>_plots.py` and its
  figures in `reports/plots/`. Delete a report's figures with the
  report. Run scripts from the repository root:
  `uv run --with matplotlib python reports/make_<slug>_plots.py $W`.
- Add a plot only when it shows the point better than a table.

## QUAIL-B plots

- One main plot over all queries and one plot per dataset, each
  including every query; no separate single-query figures.
  `reports/make_quailb_comparison_plots.py` makes them all.
- Save them as vector PDFs (`reports/plots/quailb_main.pdf`,
  `quailb_<dataset>.pdf`) with embedded fonts; commit the PDFs only,
  and link them from the report.
- Use grouped bars, readable page sizes, and one page per metric group.
  Keep method order, colors, and metric definitions the same in every
  plot.
- Show the speed-of-light estimate as a horizontal line across each
  query's bars in latency and token plots. Label it as an estimate,
  give its KV capacity and survivor assumptions, and never give it an
  accuracy. It is the distinct-prefix estimate: every shared prefix
  computed once across requests, documents, and repeated aliases.
  Check the query definitions and corpus before reusing an estimate;
  recompute stale ones on the CPU from saved inputs.
- For each query and configuration, show:
  - latency in seconds, without model startup or result collection;
  - recomputed KV tokens (`regret_tokens`): fresh tokens minus the
    fewest input tokens the requests need with unlimited KV (each
    document once, each question tail and anchor frame once per
    document, each partner suffix once after its anchor). quail-bench
    computes it (`quail_b.minimum`) from saved answer tables after the
    run; a run without it shows as not measured, never as zero;
  - fresh tokens (`fresh_tokens`): input token positions run through a
    forward pass instead of read from KV, counting repeats. Recomputed
    tokens are part of fresh tokens;
  - accuracy as agreement with the saved reference labels, naming the
    reference model, plus final output precision and recall;
  - input document counts per alias before filters, with survivor
    counts labeled separately.
- Reuse saved results unless asked to rerun, and name the source run.
  Leave out measurements from an older query definition and label
  missing baselines; never show a missing value as zero.
- When the layout changes, delete the old figures and scripts and
  update every reference.

## Other figures

- 300 DPI PNG, committed; no SVG.
- Use `reports/quail.mplstyle`
  (`plt.style.use(Path(__file__).parent / "quail.mplstyle")`) and the
  colors in `reports/plot_colors.py`.
- Every mark encodes data: no background fills, 3D, or decorative
  grid lines.
- Label data directly; use a legend only when labels would overlap.
- In profiling plots, keep the trace's own function names, such as
  `vllm.scheduler.schedule`.
- Every axis label is the unit alone, such as "microseconds per fresh
  token"; other context goes in the title. Every plot has a title,
  naming the query when it shows one.
- Put compared things side by side on one axis, and write the
  difference next to them.
- One color per category. Use a log scale only across more than one
  order of magnitude, and say so on the axis.
- Leave room between panels, keep labels the same across panels, and
  do not repeat the title in bar or tick labels.

# Issues and pull requests

- Include a figure when it shows the point better than text: a link
  to the saved run or to a plot on the report branch, pinned to a
  commit, for numbers; a mermaid diagram for a design or dataflow
  change.

## PR description format

Write for a reviewer who has not followed the conversation. Explain
the problem, the reasoning behind the solution, and the evidence
needed to assess it. Describe the final change, not the history of
implementing it. Use these sections in order, omitting sections that
add nothing for a small change:

- **Problem.** State what is broken, missing, or unnecessarily costly.
  Give a concrete example when it helps explain the need.
- **Solution.** Explain the resulting behavior and the approach. For
  a design or dataflow change, include a Mermaid diagram with precise
  labels. If a decision uses a cost model, give the actual comparison,
  define its inputs and units, and state its assumptions and omitted
  costs. "Does sharing save time?" is not a sufficient explanation.
- **Precedents.** Link to relevant existing designs and implementations.
  Explain what Quail shares with them, what differs, and why the
  differences fit the task. Using the same approach is fine. Do not
  invent novelty or claim an advantage without evidence. For prefix
  caching, discuss vLLM Automatic Prefix Caching and SGLang
  RadixAttention, with links to their documentation and code.
- **Scope and design decisions.** Explain important choices and
  tradeoffs. Identify cleanup or other changes beyond the main
  behavior. Keep each PR focused on a coherent change; recommend
  separating unrelated work instead of hiding it in the description.
- **Impact and risks.** Show the useful before-and-after evidence and
  its source. State the configurations being compared. Keep unresolved
  correctness or accuracy differences visible, and distinguish a
  possible explanation from a demonstrated cause.
- **Testing.** State what was checked, the results, and what those
  checks establish. Distinguish checks run on the current change from
  results copied from earlier runs. Do not imply that passing tests
  establish more than they cover.
- **Limits and follow-ups.** State remaining limitations and what
  evidence or work would resolve open questions.
- **Review order.** For a large change, give a short path through the
  relevant code with links and concrete questions for the reviewer.

- Use established terminology throughout the description. Define it
  where needed; do not add a terminology section or rename an existing
  technique to make Quail sound different.
- Keep the main description concise. Put long measurement tables and
  supporting details behind links or in a collapsible section.
- A clearer description does not make a large, mixed change small.
  Prefer focused changes that can be reviewed and integrated promptly.

## Guidance behind the format

- [Why we need pull request descriptions, and how to craft them][pr-writing]
  explains why descriptions should preserve intent, decisions, risks,
  testing, and limits.
- Martin Fowler's [Pull Request][fowler-pr] discusses coherent
  contributions, small PRs, and timely review.
- Fowler's [Patterns for Managing Source Code Branches][fowler-branches]
  explains frequent integration and the cost of delaying it. These
  articles guide scope and review practice; they do not prescribe the
  section template above.

[pr-writing]: https://wsbctechnicalblog.github.io/pull-request-descriptions-empowered-by-engineering-practices.html
[fowler-pr]: https://martinfowler.com/bliki/PullRequest.html
[fowler-branches]: https://martinfowler.com/articles/branching-patterns.html

---
> Source: [fsdatalab/quail](https://github.com/fsdatalab/quail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
