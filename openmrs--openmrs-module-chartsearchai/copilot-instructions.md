## openmrs-module-chartsearchai

> - When a test fails, fix the pipeline/production code — never weaken assertions, revert test data, or change expected values to match wrong behavior.

# Rules

- When a test fails, fix the pipeline/production code — never weaken assertions, revert test data, or change expected values to match wrong behavior.
- Prefer root-cause fixes over incremental symptom patches. Diagnose *why* something is broken before proposing a fix. Start with the best solution, not the quickest.
- Do not go in circles analyzing a problem without fixing it.
- Tests must call the actual production pipeline — no simulations, mocks, or reimplementations of pipeline logic in test code. This includes unit tests for internal methods: do not call internal methods directly with hand-crafted inputs. Every test must exercise the full production code path using real data (e.g. the ONNX embedding model and patient test datasets).
- When a multi-step pipeline exists (e.g. normalize → transform → embed), tests must call the composed production method, not manually chain the individual steps. Manually chaining steps in tests can mask bugs where the production code assembles the pipeline differently. If no composed method exists, extract one and use it from both production and test code.
- Tests are the specification. Modifying tests to make them pass is changing the spec — only fix the production code to satisfy the existing tests.
- Before presenting changes, review your own diff multiple times. On each pass, check every item below. Keep reviewing until a pass finds nothing to fix.
  1. Does the change match the plan?
  2. Is the logic correct — edge cases, off-by-one errors, null/empty handling?
  3. Are there any regressions — does existing behavior stay intact?
  4. Is anything missing — incomplete implementation, forgotten call sites?
  5. Does it use the real production pipeline, not a simulation or mock?
  6. Were any tests modified? If so, that's a rule violation — revert and fix the production code instead.
  7. Were all rules in this file followed?
- Follow test-driven development: for every bug fix or new feature, first write a failing test that defines the expected behavior, then write production code to make it pass.
- Always create a plan before writing code. Read the relevant code, outline the approach, then implement.
- Never commit code with known regressions. If a change improves one query but breaks another, the root cause is in shared infrastructure (e.g. noise profile, score distribution), not in the individual pipeline stage. Diagnose the shared infrastructure problem before attempting fixes — patching individual stages will keep regressing.
- When modifying the retrieval pipeline, run the cross-query regression tests (`enriched_*` in `LlmInferenceServiceTest`) AND the eval harness (`EnrichedRetrievalEvalTest`, 485 cases) before pushing. These catch cross-query side effects where improving one query silently breaks another.
- When adding concept-set category hints to new record types, also add the corresponding entries to the `*_DATASET_CATEGORY_HINTS` maps in `TestDatasetHelper` and regenerate the eval baseline. The eval baseline must always match the current hints configuration.

# API surface rules — do not bypass these methods

These methods are the ONLY correct entry points for their respective operations. Do not reimplement their logic inline, call their internal helpers directly, or hardcode their output values.

- **Prefixed text**: Use `ChartSearchAiUtils.buildPrefixedText(resourceType, text)`. Never call `getEmbeddingPrefix()` directly (it is private) or hardcode prefix strings like `"Clinical observation: "`.
- **Query embedding input**: Use `LlmInferenceService.prepareEmbeddingInput(question, queryPrefix)`. Never manually chain `stripQueryStopwords` → `buildEmbeddingQuery` → embed.
- **Retrieval pipeline**: Use `LlmInferenceService.findSimilar()`. Never reimplement scoring/ranking/filtering by calling `cosineSimilarity`, `computeKeywordScore`, or `extractQueryTerms` in a loop.
- **Building embeddings**: Use `EmbeddingIndexer.buildEmbeddings()`. Never construct `ChartEmbedding` objects manually with `provider.embed()`.
- **Cosine similarity**: Use `ChartSearchAiUtils.cosineSimilarity()`. Never reimplement the formula.
- **Test datasets**: Use `TestDatasetHelper.FULL_PATIENT_DATASET`, `TestDatasetHelper.SECOND_PATIENT_DATASET`, `TestDatasetHelper.toSerializedRecords()`, `TestDatasetHelper.inferResourceType()`, `TestDatasetHelper.stripDatasetPrefixAndDate()`. Never duplicate these helpers in individual test files.
- **Category hints in tests**: Use `TestDatasetHelper.FULL_DATASET_CATEGORY_HINTS` (and per-dataset variants). Never duplicate hints maps in individual test files.
- **Stripping category hints**: Use `ChartSearchAiUtils.stripCategoryHints()`. Never reimplement the hint-detection pattern.
- **Noise profile with enrichment**: Use `ModelNoiseProfile.compute(embeddings, provider)` (2-arg form) so cross-concept similarity is computed from hint-stripped re-embeddings. The 1-arg form without provider is a backward-compatible fallback that uses enriched vectors and will give inflated statistics when hints are present.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openmrs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
