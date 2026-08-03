## adclaim

> AdClaim answers marketing-compliance questions ("Do influencers have to disclose paid

# AGENTS.md — AdClaim

## Project summary
AdClaim answers marketing-compliance questions ("Do influencers have to disclose paid
posts?", "Can I say 'clinically proven'?") with a grounded, cited answer drawn only from
official public-domain US FTC advertising guidance, or a clean refusal when the question is
out of scope. The portfolio point is the eval: a faithfulness-verifier agent (CrewAI) is
measured, via Ragas plus custom refusal metrics, against a plain retrieve-then-answer
baseline to show it measurably cuts ungrounded answers and improves refusal accuracy.

## Stack
Python 3.13, CrewAI (orchestration), Chroma (local vector store), an embedding model
(OpenAI `text-embedding-3-small` or an open alternative), one or two chat LLMs, Ragas (RAG
eval), pytest, python-dotenv. No UI.

Full spec: `docs/agent-handoff.md`. Read it before resuming work.

## Phase checklist
- [x] Task 0 — continuity setup (this file, git init)
- [x] Phase 1 — scaffold + hygiene
- [x] Phase 2 — corpus ingest + index
- [x] Phase 3 — CrewAI crew
- [x] Phase 4 — eval harness
- [x] Phase 5 — README

## Resuming work (any agent)
1. Read `AGENTS.md` (this file) and `docs/agent-handoff.md` (full spec).
2. Run `git log --oneline` and `pytest` to see what's done and what's green.
3. Check `docs/sprint-handoff.md` for the active sprint contract (agent-agnostic:
   Claude Code, Codex, or any coding agent); resume there, else at the first unchecked
   phase above.

## Macro planning docs (managed by the Cowork macro layer; do not edit from sprints)
- `docs/planning/product-loop-map.md` — chunk sequence + status
- `docs/planning/loops/` — canonical loop specs
- `docs/planning/operating-context.md` — two-layer workflow conventions
- Sprint evidence goes to `docs/sprints/output/<sprint>-output.md`.

## Notes / API surprises
- Confirmed installable versions (Python 3.13): crewai 1.15.2, chromadb 1.1.1, ragas 0.4.3,
  openai 2.45.0, langchain-openai 0.3.35, langchain-core 0.3.86, langchain 0.3.30,
  langchain-community 0.3.27, langchain-text-splitters 0.3.11. Ragas 0.4.3 still imports
  `langchain_community.chat_models.vertexai`, removed in langchain-community 0.4.x, so the
  whole langchain-* stack is pinned to the last mutually compatible 0.3.x line. crewai has no
  langchain dependency (uses litellm), so this pin only constrains ragas's evaluator LLM
  wrapper.
- Confirmed CrewAI API (1.15.2): `Agent`/`Task`/`Crew`/`Process.sequential`, custom tools via
  `crewai.tools.BaseTool` subclass with `args_schema`, structured task output via
  `Task(output_pydantic=...)`.
- Confirmed Ragas API (0.4.3): `from ragas import evaluate, SingleTurnSample,
  EvaluationDataset`; `from ragas.metrics import Faithfulness, AnswerRelevancy,
  ContextPrecision, ContextRecall` instantiated with `llm=LangchainLLMWrapper(ChatOpenAI(...))`.
  This path is deprecated in favor of `ragas.metrics.collections` (which wants an
  `InstructorBaseRagasLLM` + explicit embeddings) but is still the documented, working
  pre-1.0 API and is what Phase 4 uses.
- Confirmed Chroma API (1.1.1): `chromadb.PersistentClient(path=...)`,
  `get_or_create_collection`, `add`/`upsert`/`query` with explicit `embeddings=` /
  `query_embeddings=` (we compute OpenAI embeddings ourselves rather than using Chroma's
  built-in embedding function).
- Phase 2 corpus already downloaded to `data/corpus/` (8 FTC documents + `manifest.json`,
  retrieved 2026-07-10) ahead of the Phase 1 close-out sprint that formally gates it; left
  uncommitted per that sprint's out-of-scope rule ("corpus downloads") rather than folded into
  the Phase 1 commit. The next Phase 2 loop should pick these files up as-is instead of
  re-downloading. One provenance wrinkle worth knowing: the first Endorsement Guides PDF found
  by search was the 2022 *proposed* rule, not the current one; the manifest correctly points at
  `p204500_endorsement_guides_in_2023.pdf`, the adopted final rule.
- Phase 2 complete: `python -m adclaim.ingest` builds 1,012 chunks into Chroma collection
  `adclaim_ftc_guidance` with stable IDs (`<doc-slug>#<chunk-idx>`), explicit OpenAI
  `text-embedding-3-small` embeddings, and required metadata `{source_title, source_url,
  section}` on every chunk. Per-document counts: Endorsement Guides PDF 248, Endorsement FAQ
  159, .com Disclosures 136, Advertising FAQs 90, Health Products Compliance Guidance 190,
  Green Guides 107, Made in USA 52, CAN-SPAM 30. Metadata verification reported
  `1012 chunks, 0 missing source_title/source_url/section`; the influencer disclosure smoke
  query returned the Endorsement Guides FAQ as the top hit. `data/index/` remains ignored.
- Phase 3 complete: `src/adclaim/crew.py` builds a sequential crew (planner, retriever with
  the `ftc_guidance_search` BaseTool, answerer, and an optional faithfulness verifier) via
  `build_crew(question, use_verifier=...)`; `python -m adclaim.ask "question"` prints the
  `AskResult` JSON (`answer`, `citations[]` of `{source_title, section}`, `refused`) to stdout
  and `verifier: on|off` to stderr, non-zero exit on failure. The refusal contract is enforced
  in plain Python, not by the LLM's self-report: the verifier task outputs a separate
  `VerifierVerdict` (`claims: [{claim, supported}]`, `out_of_corpus_scope`), and
  `agents/models.py::apply_verifier_verdict` deterministically overwrites the answer with
  `config.REFUSAL_MESSAGE` and `refused=True` if any claim is unsupported or the verdict flags
  out-of-corpus-scope; in baseline mode the answerer sets `refused` itself when retrieval
  supports nothing. Constructing `Agent`/`Task`/`Crew` objects (no `.kickoff()`) requires no
  API key or network call, which is what Task 5's tests rely on to stay network-free.
  API surprise: on a machine with no prior CrewAI run, `crew.kickoff()` prints a one-time
  interactive "Tracing Preference Saved" `rich` panel straight to stdout (triggered by
  `crewai`'s own first-run telemetry-consent flow, independent of the `CREWAI_TRACING_ENABLED`
  env var, which does not suppress it), which corrupted the JSON `adclaim.ask` writes to
  stdout on a simulated fresh install. Fixed in `config.py` by calling
  `crewai_core.user_data.update_user_data({"trace_consent": False, "first_execution_done":
  True})` at import time, the exact same call the shipped `crewai traces disable` CLI command
  makes; `crewai_core` is already an installed dependency of pinned `crewai==1.15.2`, not a new
  one. Verified by deleting the per-app user-data file (`~/Library/Application
  Support/AdClaim/.crewai_user.json` on macOS) to simulate a first-ever run and confirming
  clean JSON stdout afterward.
- Phase 4 complete: `eval/questions.yaml` (29 answerable + 11 unanswerable, every
  `expected_source` verified against `data/corpus/manifest.json`); `src/adclaim/crew.py` gained
  an additive `run_ask_with_contexts(question, use_verifier=...)` returning
  `(AskResult, contexts: list[str], token_usage: dict)` by capturing chunks on the
  `RetrieverTool` instance and reading `CrewOutput.token_usage` from `crew.kickoff()` -- `run_ask`
  and `adclaim.ask`'s behavior/schema/stdout are unchanged. `src/adclaim/eval/` holds
  `questions.py` (load+validate), `runner.py` (runs both configs, caches per-question raw
  records to `results/raw/<config>/NNN.json`, resumable), `metrics.py` (Ragas
  faithfulness/answer_relevancy/context_precision/context_recall on the answerable set via the
  documented `evaluate(dataset, metrics=[...], )` pre-1.0 API; custom refusal_accuracy/
  false_answer_rate/P50 latency/approx. cost from token usage), and `report.py`
  (`results/summary.json` -> `results/tables.md`, derived from the summary dict only).
  `python -m adclaim.eval` is the entry point.
  **Headline result:** baseline and verifier both hit 1.000 refusal accuracy / 0.000
  false-answer rate on the 11-question unanswerable set; baseline faithfulness (0.917) was
  *higher* than verifier (0.855) in this run, i.e. the verifier did not show the expected
  faithfulness uplift here -- reported as-is per the sprint's freeze-then-measure rule, not
  tuned away. Full figures in `results/summary.json` / `results/tables.md`.
  **API surprise (significant):** CrewAI's telemetry singleton
  (`crewai.events.event_listener.EventListener`) is constructed as a side effect of the very
  first `import crewai` in a process and exports spans to CrewAI's own hosted OTLP collector
  (`telemetry.crewai.com:4319`) -- a *different* mechanism from the Phase 3 tracing-consent
  banner fix. During the live eval sweep this collector was slow/unreachable and blocked crew
  execution for 15-17 minutes per task (confirmed not an OpenAI rate limit: response headers
  showed 29999/30000 requests remaining). Separately, Ragas's own `evaluate()` calls stalled
  similarly mid-run (self-resolved after ~5-18 min, consistent with the OpenAI SDK's
  default client-side timeout+retry firing on a wedged connection, not a code bug -- a fresh
  direct API call during the stall returned in ~1s, confirming the API itself was healthy).
  Fixed the CrewAI side in `config.py` by setting `CREWAI_DISABLE_TELEMETRY`,
  `CREWAI_DISABLE_TRACKING`, and `OTEL_SDK_DISABLED` (all `os.environ.setdefault(..., "true")`)
  at the very top of the file, before any other import -- config.py is the first `adclaim`
  module every entry point imports, so this reliably runs before `crewai` is ever imported.
  Verified: a verifier-mode question that previously hung 16+ minutes completed in 30.7s after
  the fix. The Ragas-side stalls were not separately patched (no equivalent quick env-var fix
  confirmed) but self-resolved within the sweep; if Phase 5 or a future eval run hits a
  similarly long stall, suspect the same class of issue (a wedged connection triggering the
  OpenAI SDK's own retry/timeout) before assuming a code bug -- check `x-ratelimit-remaining-*`
  headers via a fresh direct call to rule out rate limiting first.
  Runtime/cost of the measured run: ~2.5 hours wall clock (including the telemetry-hang delay
  before the fix), 80 crew runs (40 baseline + 40 verifier) + Ragas scoring on 58 answerable
  records across both configs; combined actual cost `0.0920 + 0.1040 = $0.196` (`results/summary.json`
  `.baseline.cost.approx_cost_usd` + `.verifier.cost.approx_cost_usd`), within the sprint's
  low-single-digit-dollar budget.
- Phase 5 complete: all five planned phases are now done. `README.md` is the portfolio
  deliverable, rewritten from the stub to present the measured baseline-vs-verifier results
  (Ragas + refusal + latency/cost) with an honest readout: the verifier improved context
  precision (+0.033) and context recall (+0.034) but not faithfulness (-0.062) or answer
  relevancy (-0.066), and the tied refusal accuracy/false-answer rate (1.000/0.000 in both
  configs) reflects an unanswerable set with no near-miss questions rather than a verifier
  win, called out explicitly rather than left implied. Every figure in the README is a direct
  copy of `results/summary.json` (as rounded in `results/tables.md`); `tests/test_readme.py`
  parses the README's results tables and asserts each displayed baseline/verifier/delta value
  against `results/summary.json` at its displayed precision, plus asserts no U+2013/U+2014
  characters anywhere in the file, so the README cannot silently drift from the measured
  results. No `src/`, `eval/`, or `results/` changes were needed or made. The project's
  planned build is now complete end to end: ingest, crew, eval, and a machine-checked README,
  all traceable back to `results/`.

---
> Source: [whoisbe/AdClaim](https://github.com/whoisbe/AdClaim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
