## testing-standards

> Testing standards and patterns for Arcade.dev toolkit


# Testing Standards for Arcade.dev Toolkit

## Test Organization and Categories
Follow the comprehensive testing pattern established in [test_get_audio_list.py](mdc:foundaudio/tests/test_get_audio_list.py):

### 1. NORMAL OPERATION TESTS
- Test basic functionality without filters
- Test functionality with various parameter combinations
- Test edge cases like empty results
- Verify correct data structure and content

### 2. INPUT VALIDATION TESTS  
- Test boundary conditions (min/max limits)
- Verify `RetryableToolError` is raised for invalid user input
- Test parameter validation logic

### 3. ERROR HANDLING TESTS
- Test missing configuration (secrets, environment variables)
- Verify `ToolExecutionError` is raised for system issues
- Test external API failure scenarios

## Mocking Patterns
### ToolContext Mocking
```python
mock_context = Mock(spec=ToolContext)
mock_context.get_secret.return_value = "test-secret-key"
```

### External API Mocking
- Mock `create_client` for Supabase interactions
- Mock complex query chains: `from_ -> select -> or_ -> contains -> order -> limit -> execute`
- Use detailed response mocking with realistic data structures

### Environment Mocking
```python
mock_getenv.side_effect = lambda key, default=None: {
    "SUPABASE_URL": "https://test.supabase.co"
}.get(key, default)
```

## Assertion Patterns
- Use explicit assertions with descriptive error messages
- Verify data types: `if not isinstance(result, dict): raise AssertionError(...)`
- Check structure and content separately
- Verify metadata fields match expected filters

## Test Documentation
- Include comprehensive docstrings explaining test purpose
- Use section comments: `# SETUP:`, `# EXECUTE:`, `# VERIFY:`
- Document why specific error types are expected
- Explain business logic being tested

---
> Source: [rsmets/arcade-foundaudio-toolkit](https://github.com/rsmets/arcade-foundaudio-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
