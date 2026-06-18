## file-xplor-skill

> >


# Xplor — Structured Cognition Engine

Xplor transforms **documents**, **codebases**, and **knowledge systems** into
traversable knowledge graphs — then reasons over those graphs rather than summarizing
raw text.

Traditional analysis: read text → produce text. Useful, but shallow.

Xplor analysis: extract structure → build graph → traverse relationships → answer
questions that raw text cannot. When asked "who is the key decision-maker in this
contract chain?" — traverse the entity graph. When asked "what breaks if I change
this function?" — walk the call graph. When asked "how does this therapy technique
connect to attachment theory?" — follow the wikilinks.

---

## Three Modes

### Document Mode
**Input:** PDFs, text files
**Method:** AI entity extraction
**Output:** People, organizations, locations, concepts, events, documents + their relationships

Extract every named entity and the relationships between them. The result is a
traversable graph, not a summary. Multiple documents can be merged — entities
deduplicated, connections traced across sources.

See `references/document-mode.md` for the extraction pipeline and entity schema.

### Code Mode
**Input:** GitHub repository URL (v1) or local repository path (v2/CLI)
**Method:** GitHub API + AI analysis (v1) | AST parsing with Tree-sitter (v2)
**Output:** Functions, classes, imports, call chains, modules + dependency relationships

v1 (web) uses the GitHub API to fetch repo structure and key files, then Claude
analyses architecture. v2 (CLI) uses Tree-sitter AST parsing for deep call-chain
graphs with full function/class/import extraction.

See `references/code-mode.md` for both pipeline specifications.

### Skill Graph Mode
**Input:** ZIP of markdown files with `[[wikilinks]]` and YAML frontmatter
**Method:** Structural parsing — frontmatter extraction + wikilink resolution
**Output:** Knowledge nodes, Maps of Content, claims, techniques + semantic connections

Each markdown file becomes a node. Each `[[wikilink]]` becomes an edge, resolved
with sentence-level context. MOC nodes cluster related concepts. Quality is scored
0–100 with specific, actionable issue reporting.

See `references/skill-graph-spec.md` for the full specification.

---

## Core Framework

### The Graph Data Model

All three modes write to the same canonical schema. This enables multi-domain
fusion — a code graph and a knowledge graph merged into one queryable layer.

```
GraphNode
  id:          Namespaced — "skill:cognitive-reframing" | "func:validateToken"
  kind:        document | code | skill
  type:        person | function | moc | claim | organization | class | ...
  name:        Display name
  description: One-sentence summary (required for quality score)
  domain:      Subject area: therapy | auth | legal | trading | ...
  tags:        []string
  content:     { full, sections[], preview }   ← progressive disclosure
  source:      { filePath, lineRange, document }
  metadata:    { wordCount, aliases, inDegree, outDegree }

GraphEdge
  source:      Node id
  target:      Node id
  type:        REFERENCES | CLUSTERS | EXTENDS | CONTRADICTS |
               CALLS | IMPORTS | DEFINES | RELATED_TO | CROSS_DOMAIN
  label:       Human-readable relationship
  strength:    1–5
  context:     Sentence that contained the link (wikilink/call-site context)
```

Full schema spec: `references/graph-core.md`

### Progressive Disclosure

Knowledge retrieval operates across five levels. Claude loads only what the
current task requires — never dumping full content into context.

| Level | Name | Content Loaded | ~Tokens/node |
|-------|------|---------------|-------------|
| 0 | Index | IDs + types only | 2 |
| 1 | Descriptions | + name + one-line description | 15 |
| 2 | Links | + connection list | 30 |
| 3 | Sections | + section headings + previews | 80 |
| 4 | Full | Complete content | 200–500 |

Traversal pattern:
```
1. Load Level 0 (full graph index) — what exists?
2. Load Level 1 for matching nodes — what is relevant?
3. Load Level 2 for top matches — what connects to what?
4. Load Level 3 for confirmed relevant nodes — confirm depth
5. Load Level 4 for 3–8 most relevant — deep read
6. Assemble context pack within token budget
```

Full rules and token budgets: `references/progressive-disclosure.md`

### Skill Graph Format

A skill graph is a folder of markdown files:

```markdown
---
name: cognitive-reframing
description: >
  A CBT technique for identifying and challenging distorted thought patterns.
type: technique
domain: therapy
tags: [cbt, cognition, distortions]
extends: [thought-records]
---

# Cognitive Reframing

Cognitive reframing involves examining the evidence for and against an
automatic negative thought (see [[thought-records]]).

## When to Apply
Use when the client presents with [[cognitive-distortions]] such as
catastrophizing or black-and-white thinking.
```

**Frontmatter fields:** `name`, `description`, `type`, `domain`, `tags`,
`aliases`, `extends`, `contradicts`

**Node types:** `skill` · `moc` · `technique` · `claim` · `framework` · `exploration`

**Edge types from wikilinks:**
- Default wikilink → `REFERENCES`
- Source node is type `moc` → `CLUSTERS`
- Frontmatter `extends: [target]` → `EXTENDS`
- Frontmatter `contradicts: [target]` → `CONTRADICTS`

**Maps of Content (MOCs):** Set `type: moc` in frontmatter. MOC nodes are
navigation entry points — they cluster related nodes and direct agent attention.
One MOC per 5–10 skill nodes is a healthy ratio.

### Quality Scoring

Every skill graph receives a deterministic score from 0–100.

| Issue | Penalty |
|-------|---------|
| Broken wikilink (target not found) | −10 per link |
| Missing `description` | −5 per file |
| Orphan node (no connections) | −3 per node |
| Missing `type` | −2 per file |
| Missing `domain` | −1 per file |
| Circular-only pair (A↔B, nothing else) | −2 per pair |

Bonuses (max +20): MOC coverage (+0–10), link density health (+0–10).

Full rubric and scoring code: `references/skill-graph-quality.md`

### Attention Scoring

When traversing a skill graph for a specific task, nodes are ranked by
attention score to determine traversal order and context inclusion.

Score inputs:
- Semantic similarity between node description and current task
- Connection count (degree centrality)
- MOC membership (MOC-adjacent nodes score higher)
- Recency / prior traversal frequency

Full attention scoring spec: `references/agent-intelligence.md`

---

## Analysis Workflows

Use these checklists to track progress on multi-step analyses.

### Document Analysis
```
Progress:
- [ ] 1. Extract all entities (name, type, description) from source text
- [ ] 2. Identify relationships between entities (label + strength 1-5)
- [ ] 3. Deduplicate entities by name across documents/chunks
- [ ] 4. Build graph (nodes + edges) using canonical schema
- [ ] 5. Identify the 5–10 highest-degree nodes (most-connected)
- [ ] 6. Surface key findings: clusters, isolated nodes, cross-doc patterns
```

### Code Analysis
```
Progress:
- [ ] 1. Map top-level modules and their responsibilities
- [ ] 2. Identify entry points (main files, exported functions, API routes)
- [ ] 3. Trace call chains from entry points to leaf functions
- [ ] 4. Flag high-impact nodes (many callers = high blast radius)
- [ ] 5. Identify dependency clusters and potential coupling issues
- [ ] 6. Summarize architecture in graph form: modules → functions → calls
```

### Skill Graph Analysis
```
Progress:
- [ ] 1. Parse frontmatter (name, description, type, domain, tags)
- [ ] 2. Extract all [[wikilinks]] with sentence-level context
- [ ] 3. Resolve links against node index (flag broken links)
- [ ] 4. Assign edge types (REFERENCES / CLUSTERS / EXTENDS / CONTRADICTS)
- [ ] 5. Detect MOC nodes and their cluster memberships
- [ ] 6. Compute quality score (base 100, apply penalties, add bonuses)
- [ ] 7. Report: score, specific issues with file names, fix recommendations
```

---

## Entity Type Reference

**Document:** `person` · `organization` · `location` · `concept` · `event` · `document`

**Code:** `function` · `class` · `variable` · `import` · `module` · `file`

**Skill Graph:** `skill` · `moc` · `claim` · `technique` · `framework` · `exploration`

### Entity Color Map
```javascript
const TYPE_COLORS = {
  // Document
  person: "#FF6B6B", organization: "#4ECDC4", location: "#45B7D1",
  concept: "#82E0AA", document: "#F0B27A", event: "#AED6F1",
  // Code
  function: "#61AFEF", class: "#C678DD", variable: "#E5C07B",
  import: "#56B6C2", module: "#98C379", file: "#ABB2BF",
  // Skill Graph
  skill: "#FF9F43", moc: "#EE5A24", claim: "#A3CB38",
  technique: "#FDA7DF", framework: "#9AECDB", exploration: "#7158e2",
  default: "#636e72",
};
```

---

## Reference Files

Read these when the task requires depth in a specific area. All files are one
level deep — linked directly from this file, never nested further.

### Core (read for any graph task)
| File | When to read |
|------|-------------|
| `references/graph-core.md` | Building or querying the graph data model |
| `references/progressive-disclosure.md` | Retrieving knowledge within token budgets |

### By Mode
| File | When to read |
|------|-------------|
| `references/document-mode.md` | Extracting entities from PDFs or documents |
| `references/code-mode.md` | Analyzing a codebase (v1 GitHub API or v2 AST) |
| `references/skill-graph-spec.md` | Building or parsing a skill graph |
| `references/skill-graph-quality.md` | Scoring or validating a skill graph |

### Advanced
| File | When to read |
|------|-------------|
| `references/agent-intelligence.md` | Attention scoring, traversal telemetry, multi-domain fusion |
| `references/search.md` | Hybrid BM25 + semantic search across graph nodes |
| `references/explorer-architecture.md` | Visualizing graphs in a UI |
| `references/mcp-server-spec.md` | Exposing graph to AI agents via MCP |
| `references/cli-spec.md` | CLI commands: index, mcp, skill, fuse |

---

## Example Skill Graphs

### Therapy (CBT)
```
therapy-cbt/
├── index.md (moc) ← entry point for agents
├── techniques/
│   ├── cognitive-reframing.md → [[thought-records]], [[cognitive-distortions]]
│   ├── thought-records.md → [[socratic-questioning]]
│   └── exposure-hierarchy.md → [[behavioral-activation]]
├── claims/
│   └── validation-first.md → [[grounding-techniques]]
└── frameworks/
    └── case-formulation.md → integrates all techniques
```

### Trading
```
trading/
├── index.md (moc)
├── mocs/ risk-management.md, market-structure.md, psychology.md
├── techniques/ position-sizing.md, stop-loss-strategies.md
├── claims/ risk-first-not-reward-first.md, process-over-outcome.md
└── frameworks/ expected-value.md, kelly-criterion.md
```

### Company Knowledge
```
acme-corp/
├── index.md (moc)
├── mocs/ org-structure.md, product-knowledge.md, processes.md
├── products/ widget-pro.md, pricing-model.md
├── processes/ incident-response.md, deployment-pipeline.md
└── culture/ decision-making.md, values.md
```

---
> Source: [amlitio/file-xplor-skill](https://github.com/amlitio/file-xplor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
