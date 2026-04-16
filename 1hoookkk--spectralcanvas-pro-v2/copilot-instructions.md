## spectralcanvas-pro-v2

> "globalInstruction": "You are an assistant working on SpectralCanvas Pro v2. Follow repo laws in CLAUDE.md. Plan-first: ALWAYS output a numbered plan and file list before code. NEVER modify RT audio or engine code paths. If a user asks to change protected files, respond with a plan-only proposal and ask for approval.",

{
  "globalInstruction": "You are an assistant working on SpectralCanvas Pro v2. Follow repo laws in CLAUDE.md. Plan-first: ALWAYS output a numbered plan and file list before code. NEVER modify RT audio or engine code paths. If a user asks to change protected files, respond with a plan-only proposal and ask for approval.",
  "rules": [
    {
      "pattern": "Source/audio/**",
      "instruction": "PROTECTED RT PATH. Do not edit. No allocations/locks/logging allowed. Provide plan-only diffs and unit tests in Tests/; require approval label: touch-engines-approved."
    },
    {
      "pattern": "Source/engine/**",
      "instruction": "PROTECTED ENGINE PATH. Do not edit. Provide plan-only diffs and tests; require approval label."
    },
    {
      "pattern": "Source/gpu/**",
      "instruction": "Allowed with care. Never block render loop; device-lost + WARP fallback must stay intact; use staging→CopySubresourceRegion; keep frame p95 ≤ 16.7 ms. Update tests and docs."
    },
    {
      "pattern": "Source/ui/**",
      "instruction": "Allowed. UI thread only; sample atomics/ring buffers read-only. No RT changes."
    },
    {
      "pattern": "Tests/**",
      "instruction": "Freely add/modify tests. Prefer expanding RTSafetyTest, ProcessBlockTest, GPUResilienceTest, PerfHarness JSON."
    },
    {
      "pattern": "docs/**",
      "instruction": "Keep BLUEPRINT, Developer-Notes, MIGRATION_LOG in sync with changes. Summarize perf evidence."
    },
    {
      "pattern": "CLAUDE.md",
      "instruction": "May append non-RT sections (tooling, docs). Do not relax RT/GPU laws."
    }
  ]
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/1hoookkk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
