## refactoring-agent-for-claude-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Claude Code agent system for analyzing legacy systems and planning microservices refactoring. The main orchestrator (`/refactor-system`) coordinates 18 specialized sub-agents through a sequential pipeline, producing Markdown analysis reports, API specifications, and a RyuGraph knowledge graph database.

## Quick Start

```bash
# Full pipeline (all phases in sequence)
/refactor-system ./SampleCode

# Individual phases (run after /analyze-system produces reports/01_analysis/)
/analyze-system ./SampleCode          # Phase 1: ubiquitous language, actors, domain-code mapping
/evaluate-mmi ./SampleCode            # Phase 2: MMI evaluation (requires Phase 1)
/map-domains ./SampleCode             # Phase 3: bounded contexts, context map
/design-microservices ./SampleCode    # Phase 4: target architecture, migration plan
/design-api ./SampleCode              # Phase 4.5: REST/GraphQL/gRPC/AsyncAPI specs
/design-scalardb ./SampleCode         # Phase 5: distributed transaction design
/create-domain-story --domain=Order   # Phase 6: interactive domain storytelling
/estimate-cost ./reports              # Phase 7: infrastructure + license cost

# Knowledge graph (parallel with main pipeline)
/build-graph ./SampleCode             # Build RyuGraph from Phase 1 outputs
/query-graph "注文に関連するクラス"     # Natural language or Cypher query
/visualize-graph ./reports/graph      # Mermaid/DOT/HTML output

# Utilities
/compile-report                       # Markdown → single HTML report
/render-mermaid ./reports             # Mermaid → PNG + SVG via mmdc
/fix-mermaid ./reports                # Auto-fix Mermaid syntax errors
/scalardb-sizing-estimator            # Interactive Pod/K8s/DB sizing & cost

# Orchestrator options
/refactor-system ./src --output=./custom-output/
/refactor-system ./src --domain=Order,Customer
/refactor-system ./src --analyze-only
/refactor-system ./src --skip-mmi
/refactor-system ./src --skip-stories
```

## Python Utilities

```bash
pip install ryugraph pandas markdown pymdown-extensions

python scripts/parse_analysis.py --input-dir ./reports/01_analysis --output-dir ./reports/graph/data
python scripts/build_graph.py --data-dir ./reports/graph/data --db-path ./knowledge.ryugraph
python scripts/query_graph.py --db-path ./knowledge.ryugraph --interactive
python scripts/visualize_graph.py --data-dir ./reports/graph/data --output-dir ./reports/graph/visualizations
python scripts/compile_report.py --input-dir ./reports --output ./reports/00_summary/full-report.html
```

## Architecture

### Prompt Pipeline Pattern

The system uses a "prompt-as-code" architecture: each skill is an LLM instruction document that Claude Code follows step-by-step. The orchestrator invokes skills sequentially via `Skill()` calls, with each skill reading prior phase outputs and writing its own files immediately.

```
User → /refactor-system ./SampleCode
  → LLM reads .claude/skills/refactor-system/SKILL.md
  → Invokes Skill(analyze-system) → writes reports/01_analysis/*.md
  → Invokes Skill(evaluate-mmi)   → writes reports/02_evaluation/*.md
  → Invokes Skill(map-domains)    → writes reports/03_design/domain-analysis.md, context-map.md
  → Invokes Skill(design-microservices) → writes reports/03_design/target-architecture.md, etc.
  → Invokes Skill(design-api)     → writes reports/03_design/api-*.md + api-specifications/
  → Invokes Skill(design-scalardb) → writes reports/03_design/scalardb-*.md
  → Invokes Skill(create-domain-story) → writes reports/04_stories/
  → Invokes Skill(estimate-cost)  → writes reports/05_estimate/
  → Invokes Skill(fix-mermaid)    → validates all Mermaid diagrams
  → Writes reports/00_summary/executive-summary.md

Parallel: /build-graph → parse CSVs → knowledge.ryugraph/
```

**Critical convention**: Skills must write output files immediately upon completing each step ("最後にまとめて出力しない"). Do not buffer all output until the end.

### Skill System

Skills live in `.claude/skills/{skill-name}/SKILL.md` with YAML frontmatter (`name`, `description`, `user_invocable`). Each contains step-by-step LLM instructions, tool usage guidance, and output format specifications.

Commands in `.claude/commands/{skill-name}-cmd.md` are the user-facing entry points (with `-cmd` suffix). They mirror the skill content but are formatted as Claude Code slash commands with `description` and `argument-hint` frontmatter.

The template at `.claude/templates/output-structure.md` defines the canonical file dependency graph and required sections for each output file — it acts as a contract between skills.

### Dual Invocation

- **As skill**: `Skill(analyze-system)` — used by the orchestrator or programmatic invocation
- **As command**: `/analyze-system-cmd ./src` — user-typed slash command in Claude Code
- Both resolve to the same SKILL.md instructions

### Sample Target System

`SampleCode/` contains **"Scalar Auditor for BOX"** — an enterprise audit application used as the canonical test target:

| Directory | Stack | Description |
|-----------|-------|-------------|
| `Scalar-Box-Event-Log-Tool/` | Spring Boot (Java) | Backend: event logging via ScalarDB/ScalarDL + Box API |
| `Scalar-Box-WebApp/` | React (JSX/Redux) | Frontend: auth, auditor pages, item views |
| `K8s-scalardb-cluster/` | Helm YAML | ScalarDB Cluster deployment values |
| `Documentation/` | PDF, XLSX | Design docs, DB schema definitions |

## Tool Priority for Code Analysis

1. **Serena MCP Tools** (primary — language-aware AST analysis, configured for Java in `.serena/project.yml`)
   - `mcp__serena__get_symbols_overview` — file structure and symbols
   - `mcp__serena__find_symbol` — symbol search across codebase
   - `mcp__serena__find_referencing_symbols` — reference tracking
   - `mcp__serena__list_dir` — directory traversal
2. **Glob/Grep** — pattern matching when Serena is unavailable or for non-Java files
3. **Read** — direct file content access

## Key Concepts

### MMI (Modularity Maturity Index)

4-axis evaluation: Cohesion (30%), Coupling (30%), Independence (20%), Reusability (20%). Each axis scored 0-5.

Formula: `MMI = (0.3×Cohesion + 0.3×Coupling + 0.2×Independence + 0.2×Reusability) / 5 × 100`

Levels: 80-100 (high — ready for microservices), 60-80 (medium), 40-60 (low-medium — major refactoring needed), 0-40 (immature)

### Domain Classification

Two orthogonal axes classify each domain:

**Business Structure**: Pipeline (sequential flow), Blackboard (shared data coordination), Dialogue (bidirectional interaction)

**Microservice Boundary**: Process (stateful business process, saga), Master (CRUD, data consistency), Integration (external adapters), Supporting (cross-cutting: auth, logging)

### Knowledge Graph Schema

**Nodes**: `Entity`, `UbiquitousTerm`, `Actor`, `Domain`, `Activity`, `Role`, `Method`, `File`

**Relationships**: `BELONGS_TO`, `DEFINED_IN`, `REFERENCES`, `CALLS`, `IMPLEMENTS`, `HAS_TERM`, `PERFORMS`, `DEPENDS_ON`

Data flow: `reports/01_analysis/*.md` → `parse_analysis.py` → `reports/graph/data/*.csv` → `build_graph.py` → `knowledge.ryugraph/`

## Adding New Skills

1. Create `.claude/skills/{skill-name}/SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: skill-name
   description: One-line description for skill discovery
   user_invocable: true
   ---
   ```
2. Create `.claude/commands/{skill-name}-cmd.md` with `description` and `argument-hint` frontmatter
3. Add permission entries in `.claude/settings.local.json` if the skill needs new tool access
4. If the skill produces output files, update `.claude/templates/output-structure.md` with the file dependency

## Model Assignment

Each skill's YAML frontmatter includes a `model` field (`opus`, `sonnet`, or `haiku`) that specifies the recommended model based on task complexity. The orchestrator (`/refactor-system`) passes this to each sub-agent via the Task tool's `model` parameter. Use `--model=[opus|sonnet|haiku]` to override all assignments at once.

| Skill | Model | Rationale |
|-------|-------|-----------|
| `refactor-system` | opus | Pipeline orchestration and judgment |
| `analyze-system` | opus | Deep code and design document analysis |
| `evaluate-mmi` | sonnet | Standardized evaluation rubric |
| `map-domains` | opus | DDD analysis, bounded context design |
| `design-microservices` | opus | Architecture design decisions |
| `design-api` | opus | Multi-protocol API specification |
| `design-scalardb` | opus | Distributed transaction design |
| `design-scalardb-analytics` | sonnet | Pattern-based analytics design |
| `create-domain-story` | sonnet | Structured data to narrative generation |
| `estimate-cost` | sonnet | Template-based estimation |
| `scalardb-sizing-estimator` | sonnet | Reference table-based calculation |
| `build-graph` | sonnet | Data parsing and script execution |
| `query-graph` | haiku | Query translation and result formatting |
| `visualize-graph` | haiku | Template-based output generation |
| `compile-report` | haiku | Script execution |
| `render-mermaid` | haiku | CLI tool execution |
| `fix-mermaid` | sonnet | Pattern matching with judgment |
| `init-output` | haiku | Directory creation |

## Output Structure

All analysis outputs go to `reports/` organized by phase:
- `00_summary/` — executive summary, metadata, compiled HTML report
- `01_analysis/` — system overview, ubiquitous language, actors, domain-code mapping
- `02_evaluation/` — MMI scores (overview, per-module, improvement plan)
- `03_design/` — domain analysis, context map, target architecture, API specs (`api-specifications/`), ScalarDB design
- `04_stories/` — per-domain storytelling documents
- `05_estimate/` — cost summary, infrastructure detail, sizing
- `graph/data/` — CSV files for graph construction
- `99_appendix/` — supporting materials

## External References

- [ScalarDB Documentation](https://scalardb.scalar-labs.com/docs/)
- [ScalarDB Analytics](https://scalardb.scalar-labs.com/docs/latest/scalardb-analytics/)
- [RyuGraph Documentation](https://ryugraph.io/docs/)

---
> Source: [wfukatsu/refactoring-agent-for-claude-code](https://github.com/wfukatsu/refactoring-agent-for-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
