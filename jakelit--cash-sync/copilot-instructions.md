## cash-sync

> **ALWAYS use the virtual environment for Python commands!**

# Project Testing Standards and Guidelines

## ⚠️ IMPORTANT: Virtual Environment Usage
**ALWAYS use the virtual environment for Python commands!**

- **Virtual Environment Path**: `.venv/`
- **Python Executable**: `.venv\Scripts\python.exe` (Windows) or `.venv/bin/python` (Unix)
- **Activation**: Always use the full path to the virtual environment's Python executable
- **Package Installation**: All packages are installed in `.venv/`, not globally

### Correct Usage Examples:
```bash
# ✅ CORRECT - Use virtual environment Python
.venv\Scripts\python.exe -m pytest tests/
.venv\Scripts\python.exe -c "import pandas; print(pandas.__version__)"

# ✅ CORRECT - Use enhanced check.py script
python check.py                    # Run all checks
python check.py test              # Run tests only
python check.py lint              # Run pylint only

# ❌ INCORRECT - Don't use system Python
python -m pytest tests/
python -c "import pandas"
```

### Before Running Any Python Commands:
1. **Check if `.venv/` exists** in the project root
2. **Use the virtual environment's Python executable** for all commands
3. **Never assume packages are installed globally**

## Documentation Standards Location
Our comprehensive documentation is located in `docs/`. Always reference these when working with code:

### Architecture Documentation
- **Main Architecture**: `docs/architecture.md` - Complete system architecture overview
- **Architecture Standards**: `docs/architecture-standards.md` - Centralized architectural standards
- **Architecture Decision Records**: `docs/adrs/` - Historical record of architectural decisions
- **ADR Index**: `docs/adrs/README.md` - Overview and format for ADRs

### Testing Standards
Our comprehensive testing standards are located in `docs/testing-standards/`. Always reference these when working with tests:

- **Test Plans**: Use `docs/testing-standards/test-plan-template.md`
- **Implementation**: Follow `docs/testing-standards/test-implementation-guide.md`
- **Coverage**: Meet requirements in `docs/testing-standards/coverage-requirements.md`
- **CI/CD**: Follow `docs/testing-standards/tools-and-automation/ci-cd-integration.md`
- **AI Prompts**: Use `docs/testing-standards/ai-prompts-guide.md` for consistent results

## Code Quality Standards

### Testing Requirements
- **Minimum Coverage**: See [official coverage requirements](docs/testing-standards/tools-and-automation/coverage_requirements.md)
- **Test Naming**: Use descriptive names like `test_method_name_condition_expected_result`
- **Test Structure**: Follow Arrange-Act-Assert pattern
- **Fixtures**: Use pytest fixtures for setup/teardown
- **Mocking**: Mock external dependencies, test behavior not implementation

### Test Organization
```python
# Standard test file structure
class TestClassNameInit:
    """Test class initialization."""
    pass

class TestClassNameCore:
    """Test core functionality."""
    
    @pytest.fixture
    def sample_instance(self):
        return ClassName(valid_params)

class TestClassNameEdgeCases:
    """Test edge cases and error conditions."""
    pass
```

### Required Test Categories
- **Unit Tests**: Individual component testing (`@pytest.mark.unit`)
- **Integration Tests**: Component interaction (`@pytest.mark.integration`)
- **Property-Based Tests**: Use `hypothesis` for comprehensive validation
- **Security Tests**: Input validation, boundary testing (`@pytest.mark.security`)

## File Conventions

### Test Files
- Location: `tests/` directory
- Naming: `test_[module_name].py`
- Mirror source structure: `src/banking/account.py` → `tests/test_account.py`

### Test Plans
- Location: `docs/test-plans/`
- Naming: `test_plan_[class_name].md`
- Use template from `docs/testing-standards/test-plan-template.md`

## Pytest Configuration
Always use our standard pytest configuration from `docs/testing-standards/pytest-configuration.md`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = [
    "--strict-markers",
    "--cov=src",
    "--cov-branch",
    "--cov-report=term-missing",
    "--cov-fail-under=90"
]
```

## Development Workflow

### When Creating Tests
1. **Check for existing test plan** in `docs/test-plans/`
2. **Create test plan if missing** using our template
3. **Implement tests** following our implementation guide
4. **Verify coverage** meets our requirements
5. **Run full test suite** before committing

### When Reviewing Code
- Verify tests follow our standards
- Check coverage requirements are met
- Ensure proper fixture usage
- Validate test naming conventions
- Confirm appropriate test categories are used

## AI Assistant Guidelines

### For Architecture Decisions
Always reference our architecture documentation when:
- Making architectural changes
- Adding new components
- Modifying existing interfaces
- Creating new modules
- Implementing design patterns

### For Test Creation
Always reference our testing standards documentation when:
- Creating new test files
- Writing test plans
- Implementing test methods
- Setting up fixtures
- Configuring mocking

### For Code Review
When reviewing code, check compliance with:
- Our architecture patterns and principles
- Testing implementation patterns
- Coverage requirements
- Naming conventions
- Code organization
- Quality standards

### Common Patterns
```python
# Preferred test method pattern
def test_deposit_positive_amount_increases_balance(self, sample_account):
    """Test that depositing positive amount increases balance."""
    # Arrange
    initial_balance = sample_account.balance
    deposit_amount = 100
    
    # Act
    result = sample_account.deposit(deposit_amount)
    
    # Assert
    assert result is True
    assert sample_account.balance == initial_balance + deposit_amount

# Preferred exception testing
def test_withdraw_insufficient_funds_raises_error(self, sample_account):
    """Test withdrawal with insufficient funds raises exception."""
    with pytest.raises(InsufficientFundsError) as exc_info:
        sample_account.withdraw(999999)
    
    assert "Insufficient funds" in str(exc_info.value)
```

## Project-Specific Notes

### Technology Stack
- **Python**: 3.9+
- **GUI Framework**: Tkinter (with graceful fallback)
- **Data Processing**: Pandas
- **Excel Operations**: openpyxl
- **Testing Framework**: pytest with pytest-cov
- **Mocking**: pytest-mock
- **Property Testing**: hypothesis
- **CI/CD**: GitHub Actions

### Architecture Overview
- **Layered Architecture**: Presentation, Application, Domain, Infrastructure layers
- **Abstract Base Classes**: Template method pattern for importers
- **Excel Table Storage**: Structured data storage using Excel Tables
- **Modular Design**: Clear separation of concerns across components

### Key Modules
- **Importers**: Banking operations require 95%+ coverage
- **Auto-Categorizer**: Pattern matching requires comprehensive testing
- **Excel Handler**: File operations need integration tests
- **GUI Components**: User interface requires usability testing
- **Data Processing**: Pandas operations need property-based tests

### External Dependencies
- Database: Mock using pytest fixtures
- External APIs: Mock HTTP calls
- File system: Use tmp_path fixture
- Time-dependent: Mock datetime

## Quick Reference Commands

```bash
# Run tests with coverage
pytest --cov=src --cov-report=html

# Run specific test categories
pytest -m unit
pytest -m integration
pytest -m "not slow"

# Generate test report
pytest --html=report.html

# Check coverage only
coverage report --show-missing
```

## Remember
- **Architecture First**: Always consider architectural implications of changes
- **Documentation**: Architecture decisions should be recorded in ADRs
- **Testing**: Tests are documentation - make them clear and readable
- **Behavior Testing**: Test behavior, not implementation details
- **Performance**: Keep tests fast and reliable
- **Standards**: Follow our standards for consistency across the team
- **Reference**: Reference our documentation when in doubt

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jakelit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
