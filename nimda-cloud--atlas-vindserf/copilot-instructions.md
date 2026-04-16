## atlas-vindserf

> Ensures code quality and adherence to project architecture.


# Quality Assurance Protocol

Ensures code quality and adherence to project architecture.

## ⚠️ Mandatory Environment Setup

**HIGHEST PRIORITY**: Before starting ANY development task, follow the environment setup protocol in `.windsurf/ENVIRONMENT_SETUP.md`. This ensures all tools are available and correctly configured for development.

## Quality Standards

1. **Linting**: run `ruff` and `mypy` before committing.
2. **Testing**: new modules require PyTest cases covering core logic.
3. **Documentation**: public functions must have docstrings following Google style.
4. **Security Review**: verify actions against security rules before merging.
5. **GUI UX Check**: manual test new widgets on macOS Sequoia for visual consistency.
6. **Performance Requirements**:
   - Screen/input tools: <100ms latency
   - Planning operations: <500ms latency
   - Memory operations: <200ms latency
   - Monitor and log all operation times
   - Implement fallbacks for slow operations
   - Auto-generate performance reports every 30 minutes
7. **Dependency and Resource Management**:
   - Keep dependencies minimal and pinned
   - Implement lazy loading for heavy modules
   - Monitor memory usage and implement auto-cleanup
   - Cache frequently used results with TTL
8. **Continuous Quality Control**:
   - Self-review code diff
   - Run performance benchmarks
   - Update CHANGELOG.md
   - Verify graceful degradation paths
9. **Language Consistency**: Ensure all comments, docstrings, and documentation are written in English.
10. **Code Coverage**: Maintain ≥ 90% statement coverage across tests. Failing the threshold blocks CI.
11. **Performance Regression**: Add automated benchmarks; flag any tool or function whose latency regresses by >10% versus the latest main branch.
12. **Secret Scanning**: Ensure no hard-coded credentials/API keys are committed (CI step with `gitleaks`).
13. **Docstring Coverage**: Maintain ≥ 85% public-API docstring coverage (enforced via `interrogate` or `pydocstyle`).

## Environment Requirements

All development MUST use the macOS Mac Studio M1 Max 32GB environment:
- **Hardware**: Mac Studio M1 Max with 32GB unified memory
- **OS**: macOS
- **Python**: 3.13.x (ARM64 native)
- **Virtual Environment**: `venv-macos`
- **Native Frameworks**: PyObjC, Metal, CoreML

## Language Quality Standards

1. **English-Only Code**: All source code, comments, and documentation must be in English.
   - Linters will enforce English naming conventions
   - Automated checks for non-English code will run during pre-commit

2. **Multilingual UI**: All user-facing strings must be available in Ukrainian, Russian, and English
   - Automated checks for missing translations
   - Default to Ukrainian when translation is unavailable

See `.windsurf/ENVIRONMENT_SETUP.md` for detailed setup instructions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nimda-cloud) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
