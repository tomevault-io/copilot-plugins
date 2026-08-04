## compare-image

> Image comparison system: extract frames from video, compare with query images using perceptual similarity.

# Image Compare - Project Context

## Project Overview
Image comparison system: extract frames from video, compare with query images using perceptual similarity.

## Methodology
This project follows **SDD + TDD + DDD + LLM Wiki**:

- **SDD (Specification-Driven Development)**: All features start as specs in `specs/`. No code is written without a spec.
- **DDD (Domain-Driven Design)**: Code structure mirrors the domain model. See `docs/architecture/domain_model.md`.
- **TDD (Test-Driven Development)**: Every domain behavior gets a test FIRST, then implementation.
- **LLM Wiki**: Knowledge base maintained via LLM Wiki desktop app + MCP. After significant changes, update wiki pages to reflect new knowledge.

## Paths

- **Code**: `/Users/pangjinfu/code/compare_image` (this project)
- **Wiki**: `wiki/` (in this project — entities, concepts, decisions, comparisons, etc.)
- **Schema**: `schema.md` (wiki page conventions and cross-referencing rules)
- **Purpose**: `purpose.md` (project goals and scope)
- **LLM Wiki App**: `/Users/pangjinfu/Documents/llm_wiki/image_compare/image_compare` (symlink to this project)
- **MCP Server**: `llm-wiki` (configured in `.claude/settings.json`)
  - Use `llm_wiki_set_project` to pin to `image_compare` project
  - Use `llm_wiki_read_file` to read wiki pages
  - Use `llm_wiki_search` to search across wiki knowledge

## Development Workflow

### 1. Specify (SDD)
1. Write/update URD in `specs/URD/` (user requirements)
2. Derive PRD in `specs/PRD/` (product requirements)
3. Create ADD in `specs/ADD/` (architecture/design)
4. Detail API specs in `specs/API/`

### 2. Model (DDD)
1. Identify bounded contexts and entities
2. Update `docs/architecture/domain_model.md`
3. Update wiki entity pages in `wiki/entities/`

### 3. Test (TDD)
1. Write failing tests in `tests/`
2. Run: `pytest tests/ -v`
3. Implement minimum code to pass
4. Refactor

### 4. Document (LLM Wiki)
After significant changes, update wiki pages:
- `wiki/entities/` - one page per domain entity
- `wiki/concepts/` - one page per key concept
- `wiki/decisions/` - ADRs (Architecture Decision Records)
- `wiki/comparisons/` - A vs B analysis pages
- `wiki/summaries/` - overview pages per bounded context

## Tech Stack
- **Language**: Python 3.11+
- **Testing**: pytest
- **Image Processing**: OpenCV (cv2), scikit-image
- **Similarity**: perceptual hashing (imagehash), feature matching
- **Video**: OpenCV VideoCapture
- **Package Management**: pip + pyproject.toml

## Project Structure
```
compare_image/
├── specs/              # SDD specifications
│   ├── URD/            # User Requirements Documents
│   ├── PRD/            # Product Requirements Documents
│   ├── ADD/            # Architecture Design Documents
│   └── API/            # API specifications
├── docs/               # Architecture & design docs
│   ├── architecture/   # System architecture, domain model
│   └── design/         # Detailed design docs
├── wiki/               # LLM Wiki knowledge base (version controlled)
│   ├── entities/       # Domain entity pages
│   ├── concepts/       # Concept/algorithm pages
│   ├── comparisons/    # A vs B analysis
│   ├── decisions/      # Architecture Decision Records (ADRs)
│   ├── specs/          # Spec summaries
│   ├── test-strategies/# Test strategy pages
│   └── synthesis/      # Cross-cutting summaries
├── schema.md           # Wiki page conventions
├── purpose.md          # Project goals and scope
├── src/image_compare/  # Source code (DDD structured)
│   ├── image/          # Image bounded context
│   ├── comparison/     # Comparison bounded context
│   ├── video/          # Video bounded context
│   └── shared/         # Shared kernel
├── tests/              # Test suites
│   ├── unit/           # Unit tests (mirror src/)
│   ├── integration/    # Integration tests
│   └── e2e/            # End-to-end tests
├── scripts/            # Utility scripts
└── config/             # Configuration files
```

## Running Tests
```bash
pytest tests/ -v                    # All tests
pytest tests/unit/ -v               # Unit tests only
pytest tests/unit/image/ -v         # Image context tests
pytest tests/ -k "test_compare" -v  # Tests matching pattern
```

## Code Conventions
- All domain entities are dataclasses or Pydantic models
- Repository pattern for data access
- Strategy pattern for comparison algorithms
- Type hints on all public functions
- Docstrings on all public modules/classes/functions
- Maximum function length: 30 lines
- Maximum file length: 300 lines

---
> Source: [760041425/compare_image](https://github.com/760041425/compare_image) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
