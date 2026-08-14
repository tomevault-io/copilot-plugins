## sdc-aws-sorting-lambda

> This file helps AI coding agents understand the repository structure, build/test conventions, and key architecture decisions for the **Space Weather SOC (SWSOC) AWS Lambda File Sorting Function**.

# AGENTS.md

This file helps AI coding agents understand the repository structure, build/test conventions, and key architecture decisions for the **Space Weather SOC (SWSOC) AWS Lambda File Sorting Function**.

## Project Overview

This is an AWS Lambda function that sorts Space Weather Science Operations Center files into appropriate S3 buckets based on filename parsing. The function:
- Processes S3 `Records` events or performs full incoming-bucket scans
- Routes files to instrument-specific buckets
- Integrates with Slack notifications and AWS Timestream logging
- Operates in **DEVELOPMENT** mode (only sorts `dev_` prefixed files) or **PRODUCTION** mode (sorts all files)

See [README.rst](README.rst) for detailed documentation and local testing instructions.

## Essential Commands

### Testing
```bash
# Run all tests with coverage
pytest --pyargs lambda_function/tests --cov=lambda_function/src --cov-report=html

# Build Docker container for local Lambda testing
cd lambda_function && docker build -t sdc_aws_sorting_lambda:latest .

# Run Lambda locally and test with sample event
docker run -p 9000:8080 -v "$(pwd)/tests/test_data:/test_data" sdc_aws_sorting_lambda:latest
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d @tests/test_data/test_padre_event.json
```

## Project Structure

```
lambda_function/
├── src/
│   ├── lambda.py              # Handler entry point for Lambda
│   └── file_sorter/
│       ├── __init__.py
│       └── file_sorter.py     # Core FileSorter class and handle_event() logic
├── tests/
│   ├── conftest.py            # Shared pytest fixtures (default_test_mission, use_mission)
│   ├── test_file_sorter.py    # Main test suite
│   └── test_data/             # Sample S3 event payloads
├── Dockerfile                 # Lambda container image
└── requirements.txt           # Production dependencies

lambda_function/src/file_sorter/file_sorter.py imports from swxsoc:
  - S3 operations (copy_file_in_s3, check_file_existence_in_target_buckets, etc.)
  - Config utilities (get_instrument_bucket, get_incoming_bucket, etc.)
  - Slack notifications (get_slack_client, send_pipeline_notification)
  - Timestream logging (create_timestream_client_session, log_to_timestream)
  - Utility functions (parse_science_filename)
```

## Key Technical Details

**Python Version**: 3.12 (must match AWS Lambda runtime)

**Environment Variables**:
- `LAMBDA_ENVIRONMENT` (default: `"DEVELOPMENT"`): Controls sorting behavior
  - `"DEVELOPMENT"`: Only sorts files with `dev_` prefix
  - `"PRODUCTION"`: Sorts all files
- `SWXSOC_MISSION` (default: `"hermes"` in tests): Configures mission-specific behavior

**Dependencies**:
- `swxsoc` (from git): Core library for S3, Slack, logging, config
- `moto==5.0.15`: Mocks AWS services in tests
- `pytest`, `pytest-astropy`, `pytest-cov`: Testing framework
- `ruff`: Code linting

**Linting**: Uses Ruff with specific ignores defined in [ruff.toml](ruff.toml)

## Testing Conventions

- **Fixtures**: `conftest.py` provides:
  - `default_test_mission`: Auto-applied fixture that sets `SWXSOC_MISSION=hermes` for all tests
  - `use_mission`: Fixture for tests that need a specific mission configuration
  
- **Mocking**: When testing `file_sorter.py`, patch Slack helpers in the module namespace:
  ```python
  # Patch at the import location in file_sorter.py, not slack_sdk
  with patch('file_sorter.file_sorter.send_pipeline_notification'):
      ...
  ```

- **AWS Mocking**: Use `moto` to mock S3 and other AWS services

## CI/CD Workflows

See [.github/workflows/](/.github/workflows/) for workflow definitions:
- **testing.yml**: Runs on PR, workflow_dispatch, and daily schedule; runs tests with coverage
- Coverage reports uploaded to Codecov

## Common Development Tasks

| Task | Command/Approach |
|------|------------------|
| Run tests | `pytest --pyargs lambda_function/tests --cov=lambda_function/src --cov-report=html` |
| Build Lambda image | `cd lambda_function && docker build -t sdc_aws_sorting_lambda:latest .` |
| Test Lambda locally | See README.rst section "Testing Lambda Locally" |
| Add new test | Place in `lambda_function/tests/test_*.py`; conftest fixtures auto-apply |
| Modify sorting logic | Edit `lambda_function/src/file_sorter/file_sorter.py` |
| Update dependencies | Edit `lambda_function/requirements.txt` (prod) or `requirements.dev.txt` (dev) |

## Deployment

The function is deployed as a zip file:
- **Production**: Latest GitHub release
- **Development/Testing**: Latest commit on `main` branch

## Key Decisions & Patterns

1. **Environment-based behavior**: The `LAMBDA_ENVIRONMENT` variable controls prod vs. dev sorting to allow testing with real AWS infrastructure without moving production files.

2. **Modular S3/Slack handling**: All AWS service interactions are abstracted via `swxsoc` library utilities, making this function focused on sorting logic.

3. **Comprehensive test coverage**: Tests use `moto` to mock AWS services and run without needing actual AWS credentials or S3 buckets.

4. **Docker for local testing**: Enables testing the exact Lambda runtime environment locally before deployment.

## Questions or Issues?

- For Lambda-specific questions, refer to [README.rst](README.rst)
- For swxsoc library details, see the [swxsoc repository](https://github.com/swxsoc/swxsoc)
- For CI/CD, check [.github/workflows/](/.github/workflows/)

---
> Source: [swxsoc/sdc_aws_sorting_lambda](https://github.com/swxsoc/sdc_aws_sorting_lambda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
