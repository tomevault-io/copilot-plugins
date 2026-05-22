## algo-trading

> This project follows enterprise best practices for professional algorithmic trading systems. Always adhere to these principles when writing or modifying code.

# Enterprise Best Practices — Algo-Trading (History-First, LLM-assisted)

## Project Architecture & Code Standards

This project follows enterprise best practices for professional algorithmic trading systems. Always adhere to these principles when writing or modifying code.

### 🏗️ Architecture Principles
- **Clear layering**: data (I/O) → core (business logic) → app (entrypoints)
- **Single responsibility**: Each module has one clear purpose
- **Stable interfaces**: Components communicate through well-defined contracts
- **Dependency injection**: Pass dependencies explicitly, avoid global state
- **Data immutability**: Raw data is never modified once written

### 📁 Repository Structure
```
src/
├── data/           # Data operations (fetch, processing, features)
├── config/         # Configuration management
├── utils/          # Common utilities (logging, validation, file ops)
├── core/           # Business logic (future: strategies, risk management)
└── app/            # Application layer (CLI, APIs)
```

### 🔧 Code Standards
- **Single responsibility**: Each function has one clear purpose
- **Explicit types**: Full type hints and docstrings for all functions
- **Error handling**: Comprehensive error handling with context enrichment
- **No hardcoded values**: Use configuration files for all parameters
- **Comprehensive logging**: Structured JSON logging with correlation IDs

### 📊 Data & Feature Engineering
- **No data leakage**: Features computed only on past data
- **Rolling windows**: Use proper rolling calculations, never center=True
- **Quality gates**: Implement comprehensive data validation
- **Lineage tracking**: Maintain full audit trail (load_id, source, config_hash)
- **Partitioned storage**: Use symbol/date partitioning for Parquet files

### 🧪 Testing Requirements
- **Unit tests**: Test individual functions and classes
- **Property tests**: Verify invariants and edge cases
- **Golden tests**: Fixed input/output validation
- **Coverage**: Maintain ≥80% coverage for core modules
- **No data leakage**: Ensure tests don't introduce future information

### 🚀 Performance & Scalability
- **Vectorized operations**: Use pandas/numpy vectorized operations
- **Chunked processing**: Process large datasets in manageable chunks
- **Memory management**: Monitor and optimize memory usage
- **Caching**: Implement intelligent caching for expensive computations
- **Parallel processing**: Use async/await and parallel execution where appropriate

### 🔒 Security & Compliance
- **No secrets in code**: Use environment variables or secret management
- **Audit logging**: Complete operation logging for compliance
- **Error tracking**: Comprehensive error classification and monitoring
- **Data validation**: Fail-closed behavior on quality violations

### 📝 Documentation Standards
- **Function docstrings**: Input, output, exceptions, examples
- **Architecture docs**: Clear module responsibilities and data flow
- **Runbooks**: Operational procedures and troubleshooting guides
- **API documentation**: Clear interface specifications

### 🎯 When Writing Code
1. **Start with architecture**: Plan the module structure first
2. **Follow single responsibility**: Each function does one thing well
3. **Add comprehensive logging**: Log all operations with context
4. **Implement error handling**: Graceful degradation and recovery
5. **Write tests**: Ensure functionality and prevent regressions
6. **Update documentation**: Keep docs in sync with code changes

### ❌ What NOT to Do
- Don't use global variables or mutable state
- Don't hardcode configuration values
- Don't skip error handling or validation
- Don't create functions with multiple responsibilities
- Don't ignore logging or monitoring requirements
- Don't introduce data leakage in features
- Don't skip tests for new functionality

### 🔍 Code Review Checklist
- [ ] Single responsibility principle followed
- [ ] Comprehensive error handling implemented
- [ ] Proper logging added with context
- [ ] Type hints and docstrings present
- [ ] Tests written and passing
- [ ] Configuration externalized
- [ ] No hardcoded values
- [ ] Performance considerations addressed
- [ ] Documentation updated

Remember: This is a professional trading system that must be reliable, maintainable, and auditable. Every line of code should reflect enterprise-grade quality standards.

---
> Source: [zoebbogner/Algo-Trading](https://github.com/zoebbogner/Algo-Trading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
