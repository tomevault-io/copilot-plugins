## merve

> This is a Python project for `mlserver-fastapi-wrapper` - a tool that wraps Python predictor classes into FastAPI inference APIs via YAML configuration.

# Claude Code Configuration

This is a Python project for `mlserver-fastapi-wrapper` - a tool that wraps Python predictor classes into FastAPI inference APIs via YAML configuration.

For testing - consider ../test-classifier, which contains a self-contained example and the corresponding test-scripts. You may read- and write to this folder!

## Git Commit Guidelines

**IMPORTANT**: When creating git commits, do NOT include:
- ❌ "Generated with [Claude Code]" lines
- ❌ "Co-Authored-By: Claude" lines
- ❌ Emoji markers (🤖) indicating AI assistance

Keep commit messages clean and professional, focusing on what changed and why, without meta-commentary about how the code was created.

## Project Structure
- `mlserver/` - Main package code (CLI, server, adapters, metrics)
- `examples/` - Example configurations and usage
- `tests/` - Comprehensive test suite (unit, integration, load)
- `monitoring/` - Prometheus + Grafana configuration
- `scripts/` - Utility scripts for coverage analysis
- `docs/` - **📚 Comprehensive documentation (IMPORTANT: Read and update continuously!)**
- `pyproject.toml` - Project configuration with dependencies

## Documentation

**IMPORTANT**: Always read and update the documentation in `docs/` when making changes!

### 📖 Documentation Index: `docs/INDEX.md`
The main documentation hub with links to all other docs. Start here to find any information.

### Key Documentation Files:
- **`docs/INDEX.md`** - Main documentation index with quick navigation
- **`docs/api-reference.md`** - Complete REST API documentation
- **`docs/configuration.md`** - YAML configuration guide and schemas
- **`docs/cli-reference.md`** - CLI commands for both classic and modern interfaces
- **`docs/architecture.md`** - System design and component architecture
- **`docs/deployment.md`** - Kubernetes deployment strategies
- **`docs/development.md`** - Development setup and debugging
- **`docs/examples.md`** - Complete working examples
- **`docs/observability.md`** - Metrics, logging, and monitoring
- **`docs/ainit.md`** - AI-powered initialization from notebooks
- **`docs/multi-classifier.md`** - Multi-model repository support

**When making changes**: Update relevant documentation immediately to keep it current

## Commands

### Core CLI Commands
There is an old version of the cli (in cli.py) which is obsolete - only use cli_v2.py (and the corresponding mlserver command!)

- **Serve**: `mlserver serve [config.yaml]` - Start ML server (auto-detects mlserver.yaml or config.yaml)
- **AI-Init**: `mlserver ainit notebook.ipynb` - **🤖 AI-powered initialization from Jupyter notebooks** (Auto-generates mlserver.yaml, predictor classes, and deployment files)
- **Version**: `mlserver version` - Display version and metadata information
- **Build**: `mlserver build` - Build Docker container from current project
- **Push**: `mlserver push --registry <url>` - Push container to registry
- **Images**: `mlserver images` - List built container images
- **Clean**: `mlserver clean` - Remove built container images

### Make Targets
- **Setup**: `make dev-setup` - Complete development environment setup
- **Install**: `pip install -e ".[test]"` - Install with test dependencies
- **Test**: `make test` - Run complete test suite
- **Server**: `make server-start` - Start ML server
- **Demo**: `make demo-full` - Complete demo with monitoring
- **Load Test**: `make demo-load` - Interactive load testing with live metrics
- **Monitoring**: `make demo-monitoring` - Start Prometheus + Grafana
- **Container**: `make demo-container` - Complete container build and test workflow

## Key Features
- **Process-based scaling**: `server.workers` creates separate processes (not threads) for container-friendly scaling
- **Modern configuration**: Unified `mlserver.yaml` format with classifier metadata
- **Versioned endpoints**: Clean `/v1/classifier-name/predict` API paths
- **Observability**: Prometheus metrics at `/metrics`, structured JSON logging, correlation IDs
- **Flexible input**: Supports records/JSON and ndarray input formats with auto-detection
- **Thread-safe predictions**: Configurable thread locking for model predictions
- **Containerization**: Full Docker build/push/clean workflow with auto-generated Dockerfiles
- **Dynamic loading**: Plugin architecture for any Python predictor class
- **Performance optimized**: Cached feature ordering, efficient numpy conversions
- **Comprehensive testing**: Unit, integration, and load testing with live metrics monitoring

## Testing Suite
- **Unit tests**: Core component testing (`tests/unit/`)
- **Integration tests**: API endpoint testing (`tests/integration/`)
- **Load testing**: Locust-based performance testing (`tests/load/`)
- **Live metrics demo**: Real-time metrics observation (`examples/load_test_demo.py`)
- **Monitoring stack**: Prometheus + Grafana setup (`docker-compose.monitoring.yml`)

## Dependencies
The project uses FastAPI, Uvicorn, Pydantic, prometheus-client, and ML libraries like scikit-learn and catboost. Test dependencies include pytest, locust, and httpx.

## Configuration

### Configuration Files
- **Configuration format**: `mlserver.yaml` (unified config with classifier metadata)
- **Auto-detection**: CLI automatically detects and uses mlserver.yaml

### Key Configuration Sections

#### Server Configuration
- `server.workers`: Number of processes (default: 1, container-friendly)
- `server.host/port`: Network binding configuration
- `server.cors`: CORS settings for cross-origin requests

#### Predictor Configuration
- `predictor.module`: Python module containing predictor class
- `predictor.class_name`: Predictor class name to instantiate
- `predictor.init_kwargs`: Arguments passed to predictor constructor

#### Observability Configuration
- `observability.metrics`: Enable Prometheus metrics collection
- `observability.structured_logging`: Enable JSON structured logs
- `observability.correlation_ids`: Generate request correlation IDs
- `observability.log_payloads`: Log request/response payloads (privacy consideration)

#### API Configuration
- `api.adapter`: Input format adapter (records|ndarray|auto)
- `api.feature_order`: Explicit feature ordering for records
- `api.thread_safe_predict`: Thread-safe model predictions
- `api.endpoints`: Enable/disable specific endpoints (predict, batch_predict, predict_proba)
- `api.version`: API version for endpoint routing

#### Configuration Extensions
- `classifier.*`: Classifier metadata (name, version, description)
- `model.*`: Model metadata (version, metrics, training info)
- `api.*`: API versioning and endpoint configuration
- `build.*`: Container build configuration

## Quick Demo
```bash
# Complete setup and demo
make demo-full

# This will:
# 1. Install dependencies and train model
# 2. Start ML server in background
# 3. Start Prometheus + Grafana monitoring
# 4. Run interactive load test showing live metrics
# 5. Display URLs for all services
```

## Testing

### Test Suite Management
📍 **Test Plan and Index**: See `tests/INDEX.md` for the comprehensive test suite index, coverage tracking, and evolution plan.

### Important Testing Guidelines
- **Always update `tests/INDEX.md`** when changing functionality or adding tests
- **Read `tests/INDEX.md` first** before modifying or adding tests to understand current coverage and patterns
- **Track test coverage** for all new features and maintain the coverage metrics in the index

### Current Status (2025-01-16)
- **Total Coverage**: 32% (Target: 80%)
- **Test Results**: 223 passing, 31 failing, 5 errors
- **Key Achievements**:
  - server.py: 72% coverage (up from 12%)
  - adapters.py: 93% coverage (up from 41%)
  - config.py: 94% coverage (maintained high)

### Quick Test Commands
```bash
# Run all tests with coverage
pytest tests/ --cov=mlserver --cov-report=term-missing

# Run specific test file
pytest tests/unit/test_server_unit.py -v

# Run integration tests
pytest tests/integration/ -v

# Load testing
cd tests/load && locust -f locustfile.py --host http://localhost:8000
```

### Test Organization
- `tests/unit/` - Unit tests for individual components
- `tests/integration/` - Integration tests for API endpoints and workflows
- `tests/load/` - Load testing with Locust
- `tests/fixtures/` - Shared test fixtures and mocks
- `tests/INDEX.md` - **Central test documentation and tracking**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/core64-lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
