## chronon

> We use the mill build system for this project. A couple of examples of command incantations:

# Building
We use the mill build system for this project. A couple of examples of command incantations:
- Clean the build artifacts: `./mill clean`
- Build the whole repo: `./mill __.compile`
- Build a module: `./mill cloud_gcp.compile`
- Run tests in a module: `./mill cloud_gcp.test`
- Run a particular test case inside a test class: `./mill spark.test.testOnly "ai.chronon.spark.kv_store.KVUploadNodeRunnerTest" -- -z "should handle GROUP_BY_UPLOAD_TO_KV successfully"`
- Run specific tests that match a pattern: `./mill spark.test.testOnly "ai.chronon.spark.analyzer.*"`
- List which modules / tasks are available: `./mill resolve _`
 
# Workflow
- Make sure to sanity check compilation works when you’re done making a series of code changes
- When done with compilation checks, make sure to run the related unit tests as well (either for the class or module)
- When applicable, suggest test additions / extensions to go with your code changes

# Code Commenting Style
- Only add comments and scaladoc that provide non-obvious context, assumptions, or implementation details
- DO NOT add comments that simply describe what the code does (e.g., "creates a table", "returns the result")
- DO NOT add scaladoc that just restates parameter names or types
- DO add comments that explain:
  - Why something is done a certain way (e.g., "Deterministic seed for repeatable comparisons")
  - Important assumptions (e.g., "tsMillis ignored by multiPut for streaming tables")
  - Non-obvious behavior (e.g., "Ordering across threads doesn't matter for percentile/mean computation")
  - Implementation details that aren't clear from the code (e.g., "READ TileKey construction: tileStartTs deliberately None")
- Keep comments concise and focused on the "why" rather than the "what"

---
> Source: [zipline-ai/chronon](https://github.com/zipline-ai/chronon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
