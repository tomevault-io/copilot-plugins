## pensieve

> Pensieve+ extends Pensieve into an agentic privacy tracing system that

# Pensieve+ — Project Context for Claude Code

## What this project is
Pensieve+ extends Pensieve into an agentic privacy tracing system that 
detects Context-Drift Violations (CDV) in LLM memory usage.
This is the implementation artifact for a NeurIPS 2026 paper submission.

## Theoretical Framework (do not change these definitions)

### TP Poset Family
Transmission Principle ordering by constraint strength:
public(0) < consent-required(1) < reciprocity(2) < confidentiality(3)

Category baselines:
- preference → public
- behavioral → consent-required  
- financial → consent-required
- medical → confidentiality
- identity → confidentiality

### CDV Definition
A Context-Drift Violation occurs when a memory acquired in context C1 
with TP baseline T1 is used in context C2 where the effective TP T2 
has lower rank than T1. Severity: gap=1 → warning, gap>=2 → critical.

## Module Status
- Module 1 (Theory): COMPLETE — lib/cdv.ts
- Module 2 (Related Work): NOT STARTED
- Module 3 (Benchmark): NOT STARTED  
- Module 4 (Pensieve+): IN PROGRESS — lib/memory.ts extended, API layer done
- Module 5 (Experiments): NOT STARTED

## File Map
- lib/cdv.ts → CDV detection logic (core theory implementation)
- lib/memory.ts → MemoryUnit type with info_type, origin_context, origin_tp
- lib/openai.ts → generateResponse with CDV pre-check
- app/api/query/route.ts → API route with CDV results in response
- components/MemoryCard.tsx → UI with CDV badges (in progress)

---
> Source: [DrJonaC/Pensieve](https://github.com/DrJonaC/Pensieve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
