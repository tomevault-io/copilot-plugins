## test-duckdb

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DuckDB Vector Search benchmarking project focused on systematically analyzing the performance characteristics of DuckDB's VSS (Vector Similarity Search) extension for text vector search scenarios. The project follows a **functional programming paradigm** with immutable data structures, pure functions, and explicit effect handling.

## Architecture

### Functional Programming Design
The project uses a layered functional architecture (see plan/ directory for detailed design):
- **Pure Layer**: Business logic with no side effects (data generation, transformations, calculations)
- **Effect Layer**: IO operations wrapped in monadic types (database, file I/O, metrics collection)
- **Pipeline Layer**: Function composition to build complex workflows from simple functions

### Experiment Matrix
48 experimental combinations testing:
- **Data Scales**: 10K, 100K, 250K vectors
- **Vector Dimensions**: 128, 256, 512, 1024
- **Search Types**: Pure vector search, Hybrid (vector + BM25)
- **Filter Conditions**: With/without metadata filtering

### Key Components
- **Database**: DuckDB with VSS extension (HNSW indexing)
- **Text Data**: Korean text generated using Faker library
- **Type System**: Immutable dataclasses with frozen=True
- **Effect Management**: IO monad for side effects, Either for error handling
- **Performance Metrics**: Throughput (QPS), response time, accuracy (Recall@K), resource usage

## Common Commands

### Environment Setup
```bash
# Install Python dependencies (recommended: use uv)
uv sync

# Alternative: pip install
pip install duckdb faker pandas numpy psutil matplotlib seaborn plotly pyrsistent

# Install and verify DuckDB VSS extension
python test_duckdb_vss_installation.py

# Verify VSS installation
python -c "import duckdb; conn = duckdb.connect(); print(conn.execute('SELECT * FROM duckdb_extensions() WHERE extension_name = \'vss\'').fetchall())"

# Check DuckDB version (VSS requires compatible version)
python -c "import duckdb; print(f'DuckDB version: {duckdb.__version__}')"

# Run tests to verify setup (87 tests, 99% success rate)
pytest tests/ -v
```

### Running Experiments
```bash
# Run all 48 experiment combinations (sequential)
python -m src.runners.experiment_runner --all

# Run all experiments in parallel (recommended)
python -m src.runners.experiment_runner --all --parallel

# Run with custom parallel settings
python -m src.runners.experiment_runner --all --parallel --workers 6 --max-memory 8000

# Run specific experiment matrix with parallel execution
python -m src.runners.experiment_runner --data-scale small --dimensions 128,256 --parallel

# Resume from checkpoint
python -m src.runners.experiment_runner --resume --checkpoint-dir checkpoints/

# Monitor experiment progress
python -m src.tools.monitor --experiment-dir experiments/

# Run tests (87 unit tests, 99% success rate)
python -m pytest tests/ -v
python -m pytest tests/pure/ -v  # Test only pure functions
python -m pytest tests/runners/ -v  # Test runners including parallel execution
python -m pytest tests/effects/ -v --db-mode=mock  # Test effects with mocks
```

### DuckDB VSS Operations
```sql
-- Create HNSW index
CREATE INDEX idx_name ON table_name USING HNSW(vector_column)
WITH (ef_construction = 128, ef_search = 64, M = 16, metric = 'cosine');

-- Vector similarity search
SELECT * FROM table_name
ORDER BY array_distance(vector_column, query_vector::FLOAT[n])
LIMIT k;

-- Hybrid search (vector + text)
WITH vector_results AS (
    SELECT id, array_distance(vector, ?::FLOAT[n]) as v_score
    FROM table_name
    ORDER BY v_score LIMIT 100
),
text_results AS (
    SELECT id, fts_main_table.match_bm25(id, ?) as t_score
    FROM table_name
    WHERE text LIKE '%' || ? || '%'
)
SELECT v.id, (0.7 * (1 - v.v_score)) + (0.3 * t.t_score) as score
FROM vector_results v
JOIN text_results t ON v.id = t.id
ORDER BY score DESC LIMIT k;
```

## Key Implementation Considerations

### Functional Programming Patterns
- **Immutability**: Use `@dataclass(frozen=True)` for all data structures
- **Pure Functions**: Separate business logic from I/O operations
- **Effect Handling**: Wrap all side effects in IO monad
- **Error Handling**: Use Either type instead of exceptions in pure code
- **Function Composition**: Build pipelines using `compose` and `pipe`

### Type System Usage
```python
# Example type definitions
Vector = NewType('Vector', List[float])
DocumentId = NewType('DocumentId', str)
ExperimentConfig = @dataclass(frozen=True)
IO[T] = Effect wrapper for side effects
Either[E, T] = Error handling without exceptions
```

### DuckDB VSS Limitations
- VSS extension is experimental, not production-ready
- Requires vectors as FLOAT[n] arrays (32-bit floats only)
- HNSW indexes must fit entirely in RAM
- Updates mark deletions only; periodic compression needed

### Performance Optimization
- Test different HNSW parameters (ef_construction, ef_search, M)
- Monitor memory usage carefully as indexes are RAM-resident
- Use parallel_map for concurrent pure operations
- **Use parallel experiment execution** with `--parallel` flag for faster benchmarking
- **Dynamic worker scaling** based on available memory and CPU cores
- Use connection pooling wrapped in Reader monad
- Profile queries with EXPLAIN ANALYZE

### Korean Text Data Generation
- Use Faker with Korean locale (`ko_KR`)
- Generate with deterministic seeds for reproducibility
- Text categories: news articles, product reviews, documents, social media
- Include metadata for filtering tests (category, timestamp, user_id)

## Project Structure

```
src/
├── types/              # Type definitions (frozen dataclasses)
├── pure/               # Pure functions (no side effects)
│   ├── generators/     # Data generation
│   ├── transformers/   # Data transformations
│   └── calculators/    # Metrics and analysis
├── effects/            # Side effect management
│   ├── db/            # Database IO operations
│   ├── io/            # File IO operations
│   └── metrics/       # Performance monitoring
├── pipelines/          # Function composition pipelines
└── runners/            # Main entry points
    ├── experiment_runner.py    # CLI experiment runner with parallel support
    ├── parallel_runner.py      # Parallel execution engine (Phase 4B)
    ├── checkpoint.py           # Checkpoint management
    └── monitoring.py           # Resource monitoring

plan/                   # Functional design documentation
├── 01-functional-architecture.md
├── 02-type-definitions.md
├── 03-pure-functions.md
├── 04-effect-management.md
├── 05-pipeline-composition.md
├── 06-experiment-workflow.md
├── 07-implementation-guide.md
└── 08-current-status.md        # Current implementation status
```

## Experiment Workflow

The project follows a structured experiment workflow with checkpointing:

1. **Data Generation**: Korean text with deterministic seeds for reproducibility
2. **Database Initialization**: Create tables and load VSS extension
3. **Data Insertion**: Batch insertion with performance metrics
4. **Index Building**: HNSW index with dimension-optimized parameters
5. **Search Execution**: Pure vector or hybrid search with filtering
6. **Result Analysis**: Calculate accuracy metrics and generate reports

Each stage supports checkpointing for resumability. See `plan/06-experiment-workflow.md` for detailed workflow design.

## Key Functions and Pipelines

### Pure Functions (src/pure/)
- `generate_vector(seed, dimension)`: Create normalized vectors deterministically
- `calculate_recall_at_k(retrieved, relevant, k)`: Accuracy metrics
- `batch_documents(documents, batch_size)`: Split data for processing

### Effect Functions (src/effects/)
- `with_db_connection(config, f)`: Managed database connections
- `measure_io(io)`: Performance measurement wrapper
- `parallel_map_io(f, items)`: Concurrent IO execution

### Pipeline Composition (src/pipelines/)
- `single_experiment_pipeline(config)`: Complete experiment execution
- `data_preparation_pipeline(config)`: Generate test data
- `analysis_pipeline(results)`: Aggregate and visualize results

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cagojeiger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
