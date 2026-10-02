## sap-hana-client-generator

> - Install package: `pip install -e .`

# SAP HANA Client Generator Development Guide

## Build and Test Commands
- Install package: `pip install -e .`
- Run all tests: `python -m unittest discover -s tests`
- Run specific test: `python -m unittest tests.test_generator.TestClientGenerator.test_sanitize_name`
- Install and test: `./install_and_test.sh`
- Generate example client: `python example.py`

## Code Style Guidelines
- **Imports**: Standard library first, then third-party, then local modules (alphabetically sorted)
- **Formatting**: 4-space indentation, 100 character line length max
- **Type Annotations**: Use Python type hints for all function parameters and return values
- **Naming Conventions**: 
  - Classes: PascalCase (e.g., `ClientGenerator`)
  - Functions/methods: snake_case (e.g., `sanitize_name`)
  - Private methods/variables: Prefix with underscore (e.g., `_load_spec`)
- **Error Handling**: Use specific exceptions with descriptive messages
- **Documentation**: Use docstrings for all public methods/functions and include type info
- **Tests**: Write unit tests for all functionality with appropriate mocking

---
> Source: [bikash119/sap_hana_client_generator](https://github.com/bikash119/sap_hana_client_generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
