## benchmark-report

> Rules for running js-framework-benchmark and generating reports


# Benchmark Report Rules

When running `scripts/update-js-framework-benchmark-report.mjs`:

1. **NEVER pass `--framework` when generating the report.** The script reads all frameworks from `benchmark-report.config.json`. Passing `--framework` overrides that list and produces a report with only the specified framework, which is WRONG.

2. To **run + rebuild only gea**, then **generate the full report**:
   ```bash
   # Step 1: run benchmarks for gea only
   node scripts/update-js-framework-benchmark-report.mjs --run --rebuild --framework keyed/gea

   # Step 2: regenerate the full report with ALL frameworks from config
   node scripts/update-js-framework-benchmark-report.mjs
   ```

3. The final report generation (step 2) must ALWAYS be run **without** `--framework` so that all configured frameworks appear in the HTML report.

---
> Source: [dashersw/gea](https://github.com/dashersw/gea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
