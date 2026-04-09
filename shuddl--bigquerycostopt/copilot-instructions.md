## bigquerycostopt

> - Build: `pip install -e .` or `pip install -e ".[dev,test]"`

# Claude Coding Assistant Guidelines

## Commands

- Build: `pip install -e .` or `pip install -e ".[dev,test]"` 
- Lint: `flake8 bigquerycostopt tests`
- Format: `black bigquerycostopt tests` and `isort bigquerycostopt tests`
- Type check: `mypy bigquerycostopt`
- Test all: `pytest` or `./run_tests.sh`
- Test specific: `pytest tests/path_to_test.py::TestClass::test_name`
- Test groups: `pytest tests/unit/` or `pytest tests/integration/`
- Performance tests: `./run_performance_tests.sh`

## Code Style

- Python 3.9+ required
- Follow PEP 8 for formatting
- Use Black formatter with 88 character line length
- Use type hints for all function parameters and return values
- Import order: standard library, third-party, local packages (use isort)
- Name classes in CamelCase, functions/variables in snake_case
- Use descriptive variable names - avoid abbreviations
- Handle errors explicitly with try/except - log all exceptions
- Use f-strings for string formatting
- Document all public functions and classes with docstrings
- Use dataclasses for data containers

## Project Overview

The BigQuery Cost Intelligence Engine (BCIE) is designed to analyze large BigQuery datasets and provide actionable cost-saving recommendations. It integrates with Retool dashboards, stores recommendations in BigQuery, and leverages machine learning for enhanced insights.

## Architecture Guidelines

- **Serverless Focus:** Utilize serverless GCP components (Cloud Run, Cloud Functions, Pub/Sub)
- **Asynchronous Processing:** Employ Pub/Sub for asynchronous messaging and processing of large datasets
- **BigQuery Storage:** Store all recommendations and status information in BigQuery tables
- **API Design:** Design RESTful APIs for integration with Retool dashboards, ensure APIs are secure and authenticated
- **Modular Design:** Maintain a modular design with clear separation of concerns

## Module-Specific Guidelines

- **Data Connector:** Use appropriate authentication methods for BigQuery API access, implement efficient, paginated queries
- **Optimizer Modules:** Develop specific analysis algorithms for each optimization area, calculate potential cost savings and ROI
- **Recommendation Engine:** Standardize recommendation data structures, implement prioritization logic
- **Implementation Plan Generator:** Generate executable SQL scripts and verification queries, provide rollback procedures
- **Retool Integration Layer:** Handle webhook calls and asynchronous processing, provide status updates
- **Machine Learning Enhancement Module:** Implement feature engineering and model training pipelines, enhance recommendations

## Testing Guidelines

- **Unit Tests:** Write unit tests for all modules with high coverage
- **Integration Tests:** Create tests to verify interactions between modules
- **End-to-End Tests:** Implement tests to validate complete workflows
- **Performance Tests:** Run tests with varying dataset sizes

## Development Environment Setup

1. Ensure Python 3.9+ is installed on your system
2. Run the setup script: `./setup_dev.sh`
3. Activate the environment: `source venv/bin/activate`
4. Run tests with: `./run_tests.sh` or `pytest tests/`

### Common Issues

- If Python commands aren't found, ensure the virtual environment is activated
- For environment-specific settings, create a `.env` file in the project root
- When deploying to production, use the `deploy.sh` script

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shuddl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shuddl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
