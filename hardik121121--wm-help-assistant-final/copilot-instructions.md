## wm-help-assistant-final

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Maximum-quality RAG system for complex multi-topic queries across 2,257 pages of Watermelon documentation. **PRODUCTION READY** with 78% precision, 100% MRR, and 92.7% quality score.

**GitHub**: https://github.com/hardik121121/wm_help_assistant_final

**Key Features**:
- 2,133 AI-enhanced chunks with topics, summaries, and semantic classification
- Query decomposition for complex multi-topic questions
- Hybrid search (Vector + BM25 + RRF) with Cohere reranking
- Pre-built embeddings (60MB) and indexes (64MB) - ready to use

## Common Commands

### First-Time Setup
```bash
# After cloning
cp .env.example .env
# Edit .env with your API keys
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Running the System
```bash
# Launch Streamlit UI (most common)
./run_app.sh

# Test end-to-end pipeline
python -m src.generation.end_to_end_pipeline

# Run evaluation (5 queries recommended - Groq free tier limit)
python -m src.evaluation.comprehensive_evaluation

# Validate configuration
python -m config.settings
```

### Testing Individual Components
All modules with `if __name__ == "__main__"` can be tested independently:
```bash
python -m src.query.query_understanding
python -m src.retrieval.hybrid_search
python -m src.generation.answer_generator
python -m src.database.vector_store
python -m src.database.bm25_index
```

**CRITICAL**: Always use `python -m src.module.name` NOT `python src/module/name.py`

### Utility Scripts
Located in `scripts/` - standalone, no src/ imports:
```bash
# Compare evaluation results (A/B testing)
python scripts/compare_evaluations.py tests/results/baseline.json tests/results/new.json

# Diagnose quality issues
python scripts/diagnose_quality.py

# Enrich chunks with computed metadata
python scripts/enrich_chunks.py
```

## Critical Architecture Patterns

### 1. All Data Included in Repository

**NO external dependencies or reprocessing needed!**
- ✅ `cache/hierarchical_chunks.json` (5.2 MB, 2,133 chunks)
- ✅ `cache/hierarchical_embeddings.pkl` (60 MB, 2,106 vectors)
- ✅ `cache/bm25_index.pkl` (64 MB, 16,460 vocab terms)
- ✅ `cache/images/` (1,549 semantically-named images)

**Just clone, add API keys, and run!**

### 2. Dataclass-Based Architecture

**All major objects are dataclasses** (no ORM):
```python
# ✅ Correct - nested access
pipeline_result.answer.answer
pipeline_result.validation.overall_score

# ❌ Wrong - flattening doesn't exist
pipeline_result.answer_text  # AttributeError!

# ✅ JSON serialization
from dataclasses import asdict
result_dict = asdict(result)

# ✅ Mutable defaults
@dataclass
class MyClass:
    items: List[str] = field(default_factory=list)  # Correct
    tags: List[str] = []  # Wrong - shared between instances!
```

### 3. Flat Settings Structure

```python
# ✅ Correct
settings.pdf_path
settings.vector_weight
settings.chunk_size

# ❌ Wrong - settings are NOT nested
settings.paths.pdf_path
settings.retrieval.vector_weight
```

### 4. Pinecone Content Mapping (Critical Fix - Nov 2, 2025)

Pinecone has 40KB metadata limit. Full content/metadata loaded at initialization:

```python
# In hybrid_search.py __init__
self.chunk_content_map = {chunk['metadata']['chunk_id']: chunk.get('content', '')}
self.chunk_metadata_map = {chunk['metadata']['chunk_id']: chunk.get('metadata', {})}

# During retrieval - ALWAYS restore full data
chunk_id = match.metadata.get('chunk_id')
content = self.chunk_content_map.get(chunk_id, '')  # Full content
full_metadata = self.chunk_metadata_map.get(chunk_id, {})
merged_metadata = {**match.metadata, **full_metadata}
```

**If retrieval returns empty content, check content_map is loaded!**

### 5. Query Expansion is Automatic

Every query → 3 variations with 32 synonym mappings:
- `src/query/query_expander.py` - integration name mapping
- Applied automatically in `hybrid_search.py`
- Example: "zendesk" → ["zendesk", "zendesk support", "zendesk help desk"]

### 6. RRF Weights Configuration (Nov 7, 2025)

**50/50 proven optimal** via A/B testing:
```python
# config/settings.py
vector_weight: float = 0.5  # Semantic search (50%)
bm25_weight: float = 0.5    # Keyword search (50%)
rrf_k: int = 60

# Can override per-query in hybrid_search.py
results = hybrid_search.search(
    query="...",
    vector_weight=0.6,  # Override if needed
    bm25_weight=0.4
)
```

**Tested**: 50/50 → 78% precision, 100% MRR vs 45/55 → 72% precision, 82% MRR

### 7. Synchronous Architecture

**No async/await in codebase** (intentional):
- Simpler debugging
- Context chaining requires sequential processing
- Future: parallelization possible in Phase 9

## Module Execution Pattern

**ALWAYS use `python -m` for imports to work correctly:**
```bash
# ✅ Correct
python -m src.evaluation.comprehensive_evaluation
python -m src.generation.end_to_end_pipeline

# ❌ Wrong - import errors
python src/evaluation/comprehensive_evaluation.py
```

## Data Structure

### Enhanced Chunks (23 Metadata Fields)

```python
chunk = {
    'content': '...',  # Full text content
    'metadata': {
        # Identification
        'chunk_id': 'sec_156_chunk_2',
        'section_id': 'sec_156',

        # Navigation
        'page_start': 38, 'page_end': 40,
        'heading_path': ['Installation', 'Kubernetes', 'Configuration'],
        'current_heading': 'Configuration',
        'heading_level': 3,

        # AI-Enhanced (UNIQUE - from docling_processor)
        'content_type': 'configuration',  # troubleshooting, integration, etc.
        'topics': ['kubernetes', 'deployment', 'security'],
        'content_summary': 'LLM-generated summary...',
        'integration_names': ['Slack', 'MS Teams'],
        'technical_depth': 'high',

        # Structural Detection
        'has_code': True,
        'has_tables': False,
        'has_images': True,
        'has_lists': True,

        # Image & Table
        'image_paths': ['cache/images_enhanced/kubernetes_config_page38_img0.png'],
        'image_captions': ['Kubernetes deployment diagram'],
        'table_texts': [],

        # Chunk Management
        'is_continuation': False,
        'chunk_index': 0,
        'total_chunks_in_section': 1,
        'token_count': 282,
        'char_count': 1128,
        'is_toc': False
    }
}
```

**See**: `docs/REFERENCE_CARD.md` for complete field descriptions

## Common Errors to Avoid

### 1. Import Errors
❌ `python src/evaluation/comprehensive_evaluation.py`
✅ `python -m src.evaluation.comprehensive_evaluation`

### 2. Settings Structure
❌ `settings.paths.pdf_path`
✅ `settings.pdf_path`

### 3. Content Mapping
❌ Retrieving from Pinecone without loading content_map
✅ Load content_map in `__init__`, restore during retrieval

### 4. JSON Serialization
❌ `json.dumps(dataclass_instance)`
✅ `json.dumps(asdict(dataclass_instance))`

### 5. Mutable Defaults
❌ `items: List[str] = []`
✅ `items: List[str] = field(default_factory=list)`

### 6. Groq Rate Limits
❌ Running 30 test queries at once (exceeds free tier)
✅ Run 5 queries at a time (~14 queries/day limit)

### 7. RRF Weight Changes
❌ Changing weights without A/B testing
✅ Save baseline, make change, compare with `scripts/compare_evaluations.py`

### 8. Data Integration
❌ Trying to reprocess PDF (all data already included!)
✅ Use existing `cache/` files (ready to go)

## Architecture Overview

```
Query → Understanding → Multi-Step Retrieval → Generation → Validation
         (Phase 3)       (Phase 4)              (Phase 6)    (Phase 6)
```

**5 Layers**:
1. **Query Understanding**: Decomposition, classification, intent analysis
2. **Database**: Pinecone (vector), BM25 (keyword), embeddings
3. **Retrieval**: Hybrid search, reranking, context organization
4. **Generation**: LLM answer generation with citations
5. **Evaluation**: Retrieval + generation metrics

**Key Classes**:
- `QueryUnderstandingEngine` - Decomposes complex queries
- `MultiStepRetriever` - Independent retrieval per sub-question
- `HybridSearch` - RRF fusion of vector + BM25
- `AnswerGenerator` - Strategy-aware generation (4 strategies)
- `ResponseValidator` - Quality validation

**For complete architecture**: `docs/technical/architecture.md`

## Performance Metrics (Nov 7, 2025)

**Status**: 🟢 **PRODUCTION READY**

| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| **Precision@10** | 78.0% | >70% | ✅ Excellent |
| **MRR** | 100% | >80% | ✅ Perfect |
| **Quality Score** | 92.7% | >75% | ✅ Outstanding |
| **Completeness** | 100% | >85% | ✅ Perfect |
| **Success Rate** | 100% | >90% | ✅ Perfect |
| **Avg Query Time** | 25.2s | <30s | ✅ Fast |
| **Cost per Query** | $0.003 | <$0.01 | ✅ Cheap |

**See**: `tests/results/comprehensive_evaluation.json`

## API Keys Required

Create `.env` from `.env.example` and add:
1. **OpenAI**: Embeddings - https://platform.openai.com/api-keys
2. **Pinecone**: Vector DB - https://app.pinecone.io/
3. **Cohere**: Reranking - https://dashboard.cohere.com/api-keys
4. **Groq**: LLM (free tier) - https://console.groq.com/keys

**Groq Free Tier**: 100K tokens/day ≈ 14 queries/day (7K tokens per query)

## Cost Breakdown

**One-Time Setup** (already done - included in repo):
- Embeddings: $0.08

**Per Query**:
- OpenAI embeddings: $0.0001
- Cohere reranking: $0.002
- Groq LLM: $0 (free tier)
- **Total**: ~$0.003

**Monthly (300 queries)**: ~$10-15

## Development Workflows

### Running Incremental Evaluation
```bash
# Day 1: Baseline
python -m src.evaluation.comprehensive_evaluation  # Enter: 5
cp tests/results/comprehensive_evaluation.json tests/results/baseline.json

# Day 2: After changes
python -m src.evaluation.comprehensive_evaluation  # Enter: 5

# Compare
python scripts/compare_evaluations.py \
  tests/results/baseline.json \
  tests/results/comprehensive_evaluation.json
```

### Testing RRF Weight Changes
```bash
# 1. Edit config/settings.py
# Change vector_weight and bm25_weight

# 2. Run evaluation
python -m src.evaluation.comprehensive_evaluation

# 3. Compare with baseline
python scripts/compare_evaluations.py \
  tests/results/baseline_50_50_weights.json \
  tests/results/comprehensive_evaluation.json
```

### Adding Integration Synonyms
```python
# src/query/query_expander.py
self.integration_aliases = {
    'zendesk': ['zendesk support', 'zendesk help desk'],
    'hubspot': ['hubspot crm', 'hubspot marketing'],
    # Add new mappings here
}
```

## File Structure

```
├── config/
│   └── settings.py              # Flat Pydantic configuration
├── src/
│   ├── query/                   # Phase 3: Understanding
│   │   ├── query_understanding.py
│   │   ├── query_decomposer.py
│   │   ├── query_expander.py    # 32 synonym mappings
│   │   └── query_classifier.py
│   ├── database/                # Phase 5: Indexes
│   │   ├── vector_store.py      # Pinecone
│   │   ├── bm25_index.py
│   │   └── run_phase5.py        # One-time setup (already done)
│   ├── retrieval/               # Phase 4: Multi-step
│   │   ├── hybrid_search.py     # RRF fusion + content mapping
│   │   ├── multi_step_retriever.py
│   │   ├── reranker.py          # Cohere
│   │   └── context_organizer.py
│   ├── generation/              # Phase 6: Answer
│   │   ├── answer_generator.py  # 4 strategies
│   │   ├── smart_image_selector.py # 🆕 LLM-based image filtering
│   │   ├── response_validator.py
│   │   └── end_to_end_pipeline.py
│   └── evaluation/              # Phase 7: Metrics
│       ├── comprehensive_evaluation.py
│       ├── retrieval_metrics.py
│       └── generation_metrics.py
├── cache/                       # ✅ All data included
│   ├── hierarchical_chunks.json
│   ├── hierarchical_embeddings.pkl
│   ├── bm25_index.pkl
│   └── images/
├── scripts/                     # Standalone utilities
│   ├── compare_evaluations.py
│   ├── diagnose_quality.py
│   └── enrich_chunks.py
├── tests/
│   ├── test_queries.json        # 30 complex queries
│   └── results/                 # Evaluation outputs
├── docs/                        # Complete documentation
│   ├── technical/architecture.md
│   ├── features/smart-image-selection.md # 🆕 Smart image filtering
│   ├── guides/quick-start-ui.md  # Streamlit usage
│   ├── REFERENCE_CARD.md        # Chunk structure
│   └── INTEGRATION_GUIDE.md
├── app.py                       # Streamlit UI
├── run_app.sh                   # Launch script
└── .env.example                 # Config template
```

## Critical Non-Obvious Patterns

1. **All data included in repository** - no external dependencies
2. **Query expansion automatic** - 3 variations per query
3. **Content mapping required** - Pinecone 40KB metadata limit
4. **Dataclasses everywhere** - use `asdict()` for JSON
5. **Settings are flat** - `settings.key` not `settings.category.key`
6. **50/50 RRF weights proven optimal** - A/B tested
7. **Module execution via `python -m`** - required for imports
8. **Synchronous by design** - no async/await
9. **23 metadata fields** - vs standard 5-8 in typical RAG
10. **Groq free tier = ~14 queries/day** - batch evaluations

## Troubleshooting

### Import Errors
```
ModuleNotFoundError: No module named 'src'
```
**Solution**: Use `python -m src.module.name` not `python src/module/name.py`

### Empty Retrieval Content
```
Chunks have 0 chars, no content
```
**Solution**: Check `content_map` is loaded in `hybrid_search.py` initialization

### Settings AttributeError
```
AttributeError: 'Settings' object has no attribute 'paths'
```
**Solution**: Settings are flat - use `settings.pdf_path` not `settings.paths.pdf_path`

### Groq Rate Limit
```
Rate limit exceeded
```
**Solution**: Wait until midnight UTC or use 5 queries max per batch

### Missing Cache Files
```
FileNotFoundError: cache/hierarchical_chunks.json
```
**Solution**: Ensure you cloned from correct repo with all files

## Documentation

- **[docs/technical/architecture.md](docs/technical/architecture.md)** - Complete architecture
- **[docs/REFERENCE_CARD.md](docs/REFERENCE_CARD.md)** - Chunk structure (23 fields)
- **[docs/INTEGRATION_GUIDE.md](docs/INTEGRATION_GUIDE.md)** - Data integration details
- **[docs/setup/getting-started.md](docs/setup/getting-started.md)** - Setup guide
- **[docs/guides/quick-start-ui.md](docs/guides/quick-start-ui.md)** - Streamlit usage

## Recent Updates (Nov 7-9, 2025)

### ✅ Smart Image Selection Feature (Nov 9, 2025)
- **New File**: `src/generation/smart_image_selector.py` - LLM-based image filtering
- **Updated**: `src/generation/answer_generator.py` - Uses `_extract_images_smart()`
- **Updated**: `app.py` - Inline image display (2-column grid within answer box)
- **Updated Metrics**: Streamlit sidebar loads from `tests/results/comprehensive_evaluation.json`
- **Impact**: 15-25 images → top 6 most relevant (~85% relevance rate)
- **Cost**: +$0.0001 per query (minimal)
- **Documentation**: `docs/features/smart-image-selection.md`

### ✅ Repository Ready for Distribution
- Pushed to GitHub with all cache files
- Pre-built indexes save $0.08 and ~5 minutes setup time
- Complete documentation and evaluation results

### ✅ Integration Complete
- 2,133 AI-enhanced chunks from docling_processor
- 1,549 semantically-named images
- Verified integration (15/15 checks passed)

### ✅ Phase 5-7 Complete
- Generated embeddings: 2,106 vectors (3072-dim)
- Created Pinecone index: `watermelon-docs-v2`
- Built BM25 index: 16,460 vocabulary terms
- Evaluated 5 complex queries: 78% precision, 100% MRR

### ✅ RRF Weight Optimization
- Tested 50/50 vs 45/55 configurations
- 50/50 proven optimal (7.7% better precision)
- Made weights configurable in settings.py

## Success Criteria (Current Status)

**✅ All Phases Complete (9/9)**:
- ✅ Precision@10 > 70% (achieved 78%)
- ✅ MRR > 80% (achieved 100%)
- ✅ Quality Score > 75% (achieved 92.7%)
- ✅ Completeness > 85% (achieved 100%)
- ✅ Success Rate > 90% (achieved 100%)
- ✅ Query Time < 30s (achieved 25.2s)
- ✅ Cost < $0.01/query (achieved $0.003)

**Status**: 🟢 **PRODUCTION READY**

---

**Last Updated**: 2025-11-09
**Version**: 1.0.0
**Status**: ✅ PRODUCTION READY

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hardik121121)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hardik121121)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
