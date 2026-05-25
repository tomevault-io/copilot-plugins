## honey-comb

> This document provides setup and usage instructions for AI agents working with the Honey-Comb project.

# Honey-Comb Agent Instructions

This document provides setup and usage instructions for AI agents working with the Honey-Comb project.

## Project Overview

Honey-Comb is a context compression system for AI agent workflows. It uses rule-based and ML-based classifiers to compress context (keep essential information, remove noise) before passing it to LLMs, reducing token usage and improving performance.

## Quick Start

### Prerequisites
- Python 3.12+
- pip package manager
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/DJLougen/honey-comb.git
cd honey-comb

# Install in development mode
pip install -e .

# Install development dependencies
pip install -e ".[dev]"

# Verify installation
python -c "import honeycomb; print(honeycomb.__version__)"
```

### Run Tests

```bash
# Run all tests
pytest tests/

# Run with coverage
pytest tests/ --cov=honeycomb --cov-report=term-missing

# Run specific test file
pytest tests/test_compressor.py
```

## Project Structure

```
honey-comb/
├── honeycomb/              # Main package
│   ├── __init__.py        # Package initialization, version
│   ├── classifier.py      # ML classifier (TF-IDF + VotingClassifier)
│   ├── cli_train.py       # Training CLI
│   ├── compressor.py      # Compression rules (test output, files, reasoning)
│   ├── features.py        # Feature extraction for ML model
│   ├── firewall.py        # Main orchestrator (hot loop + cool loop)
│   ├── io.py              # JSONL I/O utilities
│   ├── labels.py          # Compression label taxonomy (CORE, DISTILL, COMPACT, DROP, STALE, ESCALATE)
│   ├── observability.py   # Metrics, logging, health checks
│   ├── session.py         # Thread-safe session state management
│   ├── budget.py          # Token budget management
│   └── config.py          # Configuration management
├── scripts/               # Utility scripts
│   ├── benchmark.py                    # Performance benchmarks
│   ├── benchmark_statistical.py        # Statistical benchmarks
│   ├── demo_pollution.py              # Demo: context compression
│   ├── demo_production.py             # Demo: production features
│   ├── generate_synthetic.py          # Generate training data
│   └── generate_visuals.py            # Generate documentation charts
├── tests/                 # Test suite (129 tests)
├── models/                # Trained ML models
├── examples/              # Example data
├── docs/                  # Documentation
├── pyproject.toml         # Project configuration
└── README.md              # Main documentation
```

## Key Concepts

### Compression Labels

Honey-Comb uses a 6-label taxonomy for context compression:

- **CORE**: Keep verbatim (system prompts, user goal, critical context)
- **DISTILL**: Extract key information (test output → summary + failures)
- **COMPACT**: Keep structure only (file → "src/foo.py (200 lines): class Foo, def bar()")
- **DROP**: Remove entirely (stale context, duplicate information)
- **STALE**: Mark for removal (superseded by newer information)
- **ESCALATE**: Pass to LLM (ambiguous content requiring understanding)

### Hot Loop vs Cool Loop

**Hot Loop** (per-message, ~1ms):
- Classifies and compresses each message as it arrives
- Uses rule-based or ML-based classification
- Deterministic compression rules per content type

**Cool Loop** (periodic, ~10-50ms):
- Runs every N turns (default: 10)
- Detects stale/superseded entries
- Enforces token budget

### Thread Safety

Honey-Comb is thread-safe by default:
- All session state operations are protected by locks
- Can be disabled for performance: `thread_safe=False`
- Use thread-safe mode in production, disable for single-threaded batch processing

### Production Features

1. **Observability**: Structured logging, metrics (Prometheus format), health checks
2. **Configuration**: Environment variables, config files (YAML/JSON)
3. **Performance Tuning**: Configurable thread safety, metrics collection, compression levels
4. **Token Budget Management**: Automatic enforcement of context limits

## Development Workflow

### Making Changes

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make changes** to code in `honeycomb/` or `scripts/`

3. **Run tests** to verify changes:
   ```bash
   pytest tests/
   ```

4. **Check linting**:
   ```bash
   ruff check honeycomb/
   ruff format --check honeycomb/
   ```

5. **Commit changes**:
   ```bash
   git add .
   git commit -m "feat: description of changes"
   ```

### Adding New Compression Rules

To add compression logic for a new content type:

1. **Add detection logic** in `firewall.py::_detect_content_type()`:
   ```python
   if self._is_new_content_type(content):
       return "new_content_type"
   ```

2. **Add compression function** in `compressor.py`:
   ```python
   def compress_new_content_type(self, content: str) -> str:
       # Implement compression logic
       return compressed_content
   ```

3. **Wire it up** in `compressor.py::compress()`:
   ```python
   if content_type == "new_content_type":
       return self.compress_new_content_type(content)
   ```

4. **Add tests** in `tests/test_compressor.py`:
   ```python
   def test_compress_new_content_type(self):
       compressor = DeterministicCompressor()
       content = "example input"
       result = compressor.compress_new_content_type(content)
       assert "expected_output" in result
   ```

### Training the ML Classifier

```bash
# Generate synthetic training data
python scripts/generate_synthetic.py --output examples/train.jsonl --count 1000

# Train the classifier
python honeycomb/cli_train.py \
  --train examples/train.jsonl \
  --eval examples/eval.jsonl \
  --output models/honeycomb.joblib

# Evaluate the model
python scripts/benchmark.py
```

### Running Benchmarks

```bash
# Basic performance benchmark
python scripts/benchmark.py

# Statistical benchmark with confidence intervals
python scripts/benchmark_statistical.py

# Generate documentation charts
python scripts/generate_visuals.py
```

### Running Demos

```bash
# Demo: Context compression (shows raw vs compressed)
python scripts/demo_pollution.py

# Demo: Production features (threading, metrics, health checks)
python scripts/demo_production.py
```

## Testing

### Test Categories

- `test_compressor.py`: Compression rules (32 tests)
- `test_firewall.py`: Core orchestrator (14 tests)
- `test_session.py`: Session management (26 tests)
- `test_classifier.py`: ML classifier (18 tests)
- `test_features.py`: Feature extraction (15 tests)
- `test_labels.py`: Label taxonomy (8 tests)
- `test_budget.py`: Token budget management (10 tests)
- `test_performance.py`: Performance benchmarks (6 tests)

### Test Commands

```bash
# Run all tests
pytest tests/

# Run with verbose output
pytest tests/ -v

# Run specific test
pytest tests/test_compressor.py::test_compress_test_output

# Run with coverage
pytest tests/ --cov=honeycomb --cov-report=term-missing

# Run only fast tests (skip performance tests)
pytest tests/ -m "not slow"
```

## Common Tasks

### Task: Add a New Configuration Option

1. **Add to config.py**:
   ```python
   @dataclass
   class HoneyCombConfig:
       new_option: bool = False  # Add with default value
   ```

2. **Add environment variable support**:
   ```python
   new_option = os.getenv("HONEYCOMB_NEW_OPTION", "false").lower() == "true"
   ```

3. **Use in code**:
   ```python
   if config.new_option:
       # Implement new behavior
   ```

4. **Add tests**:
   ```python
   def test_new_option(self):
       config = HoneyCombConfig(new_option=True)
       # Verify behavior
   ```

5. **Document in README.md**:
   ```markdown
   - `HONEYCOMB_NEW_OPTION`: Description of the option (default: false)
   ```

### Task: Add Metrics for a New Operation

1. **Define metric** in `observability.py`:
   ```python
   self.new_operation_latency = Histogram(
       "honeycomb_new_operation_seconds",
       "Time spent on new operation",
   )
   ```

2. **Record metric** in code:
   ```python
   start = time.perf_counter()
   # Perform operation
   elapsed = time.perf_counter() - start
   metrics.new_operation_latency.observe(elapsed)
   ```

3. **Export in Prometheus format** (automatic via `metrics.export_prometheus()`)

### Task: Debug Compression Issues

1. **Enable verbose logging**:
   ```bash
   export HONEYCOMB_LOG_LEVEL=DEBUG
   ```

2. **Run with debug output**:
   ```python
   import logging
   logging.basicConfig(level=logging.DEBUG)
   ```

3. **Check compression decisions**:
   ```python
   firewall = HoneyComb()
   result = firewall.process(message)
   print(f"Content type: {result.content_type}")
   print(f"Label: {result.label}")
   print(f"Compression ratio: {result.compression_ratio}")
   ```

## Performance Considerations

### Latency Targets

- **Rule-based classification**: <0.1ms per message
- **ML classification**: <5ms per message
- **Compression**: <10ms per message
- **Hot loop total**: <15ms per message
- **Cool loop**: <100ms per run

### Memory Usage

- **Typical session**: 10-50MB
- **Large session** (1000+ turns): 100-200MB
- **ML model**: ~5MB

### Optimization Tips

1. **Use rule-based classification** for latency-sensitive applications
2. **Disable metrics** in production if not needed: `metrics_enabled=False`
3. **Disable thread safety** for single-threaded batch processing: `thread_safe=False`
4. **Adjust cool loop interval** based on session length (default: 10 turns)
5. **Set token budget** to prevent context overflow: `budget=Budget(tokens=100000)`

## Troubleshooting

### Issue: Tests failing after changes

```bash
# Clear Python cache
find . -type d -name __pycache__ -exec rm -rf {} +
find . -type f -name "*.pyc" -delete

# Reinstall package
pip install -e . --force-reinstall

# Run tests again
pytest tests/ -v
```

### Issue: ML model not loading

```bash
# Check if model file exists
ls -lh models/honeycomb.joblib

# Retrain if missing
python honeycomb/cli_train.py --train examples/train.jsonl --output models/honeycomb.joblib

# Verify model loads
python -c "from honeycomb.classifier import load_model; model = load_model('models/honeycomb.joblib'); print('OK')"
```

### Issue: Unicode encoding errors in tests

The project uses ASCII-safe strings in tests to avoid Windows encoding issues. If you encounter Unicode errors:

```bash
# Set UTF-8 encoding
export PYTHONIOENCODING=utf-8

# Or run with Python UTF-8 mode
python -X utf8 -m pytest tests/
```

## Deployment Checklist

Before deploying to production:

- [ ] All tests passing: `pytest tests/`
- [ ] Linting clean: `ruff check honeycomb/`
- [ ] Benchmarks run successfully: `python scripts/benchmark.py`
- [ ] Documentation updated: README.md, AGENTS.md
- [ ] CHANGELOG.md updated with changes
- [ ] Version bumped in `pyproject.toml` and `honeycomb/__init__.py`
- [ ] Git tag created: `git tag v0.x.x`
- [ ] Pushed to GitHub: `git push origin master --tags`

## Integration with Agent Harnesses

### Hermes Integration

```python
from hermes import Agent
from honeycomb import HoneyComb

# Initialize Honey-Comb
firewall = HoneyComb(thread_safe=True, metrics_enabled=True)

# Create agent with Honey-Comb context filter
agent = Agent(model="gpt-4")
agent.set_context_filter(firewall.process)

# Agent automatically compresses context before LLM calls
response = agent.run("Fix the bug in src/auth.py")
```

### Custom Integration

```python
from honeycomb import HoneyComb

firewall = HoneyComb()

# Process each message
for message in agent_messages:
    compressed = firewall.process(message)
    # Use compressed.message for LLM context
    # compressed.metrics for monitoring
```

## Support

For issues or questions:
- GitHub Issues: https://github.com/DJLougen/honey-comb/issues
- Documentation: https://github.com/DJLougen/honey-comb#readme

## License

MIT License - see LICENSE file for details.

---
> Source: [DJLougen/honey-comb](https://github.com/DJLougen/honey-comb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
