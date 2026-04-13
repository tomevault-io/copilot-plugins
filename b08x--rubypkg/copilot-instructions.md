## rubypkg

> **Generated:** 2026-03-03

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-03
**Commit:** 6acbe27
**Branch:** development

## OVERVIEW

Ruby gem for NLP/AI applications. Integrates LLMs (RubyLLM), vector storage (pgvector), and async I/O.

## STRUCTURE

```
./
├── bin/              # Executables (ruby-pkg, setup_database)
├── config/           # Configuration files
├── db/               # Database (migrations/seeds)
├── data/             # Data files
├── lib/
│   └── ruby_pkg/     # Main library (9 .rb files)
│       └── models/   # Data models
├── implement/        # Implementation notes (plan.md, state.json)
├── notebooks/        # Jupyter notebooks
├── spec/             # Test directory (empty)
└── tmp/              # Temporary files
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Main entry | `lib/ruby_pkg.rb` | Requires all submodules |
| CLI | `bin/ruby-pkg` | Command-line interface |
| Config | `lib/ruby_pkg/configuration.rb` | Settings management |
| Database | `lib/ruby_pkg/database.rb` | DB connection |
| AI/ML | `lib/ruby_pkg/neuro_symbolic_processor.rb` | NLP processing |
| RAG | `lib/ruby_pkg/personal_rag_system.rb` | Retrieval-augmented generation |
| Cache | `lib/ruby_pkg/models/chunk_cache.rb` | Vector caching |

## CODE MAP

| File | Purpose |
|------|---------|
| `database.rb` | PostgreSQL/Sequel connection |
| `configuration.rb` | YAML config loader |
| `neuro_symbolic_processor.rb` | ML processing |
| `personal_rag_system.rb` | RAG pipeline |
| `redis_validator.rb` | Redis cache validation |
| `remote_context_generator.rb` | Context generation |
| `notebook_manager.rb` | Jupyter notebook ops |

## CONVENTIONS

- Ruby gem structure: standard bundler layout
- Module namespace: `RubyPKG` (note: also `ruby_pkg` - inconsistent)
- Database: Sequel ORM, pgvector extension
- Async: Socketry/async for I/O
- Testing: spec/ directory present but empty

## ANTI-PATTERNS (THIS PROJECT)

- No explicit test suite (spec/ empty)
- No CI/CD workflows (no .github/workflows)
- `implement/` contains planning docs mixed with code context
- Dual namespace: `lib/ruby_pkg/` + `lib/RubyPKG/` may cause confusion

## UNIQUE STYLES

- Podman for container orchestration (podman-compose.yml)
- Jupyter notebooks for experimentation
- RAG system built in-house (not using external gems)

## COMMANDS

```bash
bundle exec rake -T     # List available tasks
ruby bin/ruby-pkg        # Run CLI
ruby bin/setup_database  # Initialize DB
```

## NOTES

- Depends on: RubyLLM, Sequel, pgvector, Redis, async
- CLAUDE.md lists recommended Context7 library IDs for docs

### Use Context7 MCP for Loading Documentation

Context7 MCP is available to fetch up-to-date documentation with code examples.

**Recommended library IDs**:

- `/ankane/informers` - Postgres LISTEN/NOTIFY for Ruby
- `/chaps-io/gush` - Workflow orchestration for Ruby
- `/crmne/ruby_llm` - Unified Ruby interface to AI models (chat, image generation, embeddings)
- `/danielfriis/ruby_llm-schema` - Schema validation for RubyLLM
- `/danielfriis/ruby_llm-template` - Prompt templating with ERB and Rails integration
- `/diasks2/pragmatic_segmenter` - Rule-based sentence segmentation
- `/dry-rb/dry-schema` - Schema validation for Ruby
- `/dry-rb/dry-types` - Type system for Ruby
- `/explosion/spacy-llm` - LLM integration for spaCy NLP
- `/gjtorikian/commonmarker` - GitHub Flavored Markdown parser
- `/huggingface/sentence-transformers` - Sentence embeddings
- `/jeremyevans/sequel` - Database toolkit for Ruby
- `/pgvector/pgvector` - Vector similarity search for PostgreSQL
- `/socketry/async` - Async I/O for Ruby
- `/socketry/falcon` - Async HTTP server
- `/vicentereig/dspy.rb` - Structured LLM programming for Ruby
- `/websites/rooibos_run` - Rooibos testing framework
- `/websites/rubyllm` - RubyLLM website docs
- `/websites/spacy_io` - spaCy NLP documentation
- `/yjacquin/fast-mcp` - Fast MCP server implementation
- `/yohasebe/ruby-spacy` - Ruby bindings for spaCy
- `/seuros/breaker_machines` - Circuit Breaker pattern for Ruby (state_machines-based, thread-safe)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/b08x) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
