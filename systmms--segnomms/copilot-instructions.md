## segnomms

> This file contains important development context and reminders for Claude when working on the SegnoMMS project.

# Claude Development Context

This file contains important development context and reminders for Claude when working on the SegnoMMS project.


## ✅ Pydantic v2 + MyPy Modernization Achievement

**🎉 COMPLETED:** SegnoMMS configuration system successfully modernized to Pydantic v2 + MyPy strict typing patterns.

### Achievement Summary (as of 2025-01-25)

**Core Modernization Completed:**
- ✅ **Removed `use_enum_values=True`** - Config models now return enum objects at runtime
- ✅ **100% Config Module MyPy Compliance** - All 9 config modules pass strict MyPy validation
- ✅ **Discriminated Unions** - Shape configurations use proper type narrowing with `Field(discriminator="shape")`
- ✅ **TypedDict Patterns** - Type-safe **kwargs for shape renderers using `Unpack[TypedDict]`
- ✅ **Modern Typing Practices** - `from __future__ import annotations`, strict MyPy settings
- ✅ **Enum Objects at Runtime** - Proper enum object behavior instead of string conversion

### Technical Implementation

**Build System Updates:**
- Updated `pyproject.toml` with Pydantic `>=2.7,<3` constraint
- Added `plugins = ["pydantic.mypy"]` integration
- Enabled strict MyPy configuration with `disallow_any_generics=true`
- Added `py.typed` file for PEP 561 compliance

**Configuration Model Improvements:**
- Removed `use_enum_values=True` from all ConfigDict instances
- Added discriminated unions for shape-specific configurations:
  ```python
  ShapeSpecificConfig = Annotated[
      Union[BasicShapeConfig, RoundedShapeConfig, ConnectedShapeConfig],
      Field(discriminator="shape"),
  ]
  ```
- Updated enum comparisons to use `.value` only when string comparison needed
- Modern factory methods with proper type safety

**Type Safety Enhancements:**
- Created comprehensive TypedDict definitions in `segnomms/types.py`
- Updated shape renderers with type-safe patterns:
  ```python
  def render(
      self, x: float, y: float, size: float, **kwargs: Unpack[SquareRenderKwargs]
  ) -> ET.Element:
  ```
- Added proper type annotations throughout config modules

### Validation Status

**MyPy Compliance:**
- ✅ **segnomms/config/**: 9/9 files pass strict MyPy validation (100% clean)
- 🔄 **Overall codebase**: 215 MyPy errors remain (primarily in other modules, mostly annotation issues)

**Testing Status:**
- ✅ **Core functionality**: Configuration models work correctly with enum objects
- 🔄 **Test expectations**: Some tests still expect string values, need updating to expect enum objects
- ✅ **Backward compatibility**: Factory methods maintain compatibility with string inputs

### Best Practices Established

**For Future Development:**
1. **Always use enum objects** - Compare against `MergeStrategy.SOFT`, not `"soft"`
2. **Use discriminated unions** - For shape-specific configuration variants
3. **Apply TypedDict patterns** - For type-safe **kwargs usage
4. **Maintain strict MyPy** - All config modules must pass strict validation
5. **Test enum expectations** - Update test assertions to expect enum objects

### Achievement Impact

This modernization provides:
- **Enhanced Type Safety** - Catch more errors at development time
- **Better IDE Support** - Improved autocompletion and error detection
- **Runtime Enum Objects** - More robust enum handling without string conversion
- **Modern Pydantic Patterns** - Following current best practices
- **Discriminated Union Benefits** - Proper type narrowing for shape configurations

**The configuration system is now a model implementation of Pydantic v2 + MyPy best practices.**

---

## 🔧 Development Standards and Practices

**⚠️ IMPORTANT:** These development practices must be followed without exception.

### Git Commit Standards

**NEVER use `--no-verify` flag with git commits.**
- Pre-commit hooks exist to maintain code quality and consistency
- All formatting, linting, and type checking issues must be resolved before committing
- If pre-commit hooks fail, fix the underlying issues rather than bypassing them
- This ensures consistent code quality across all contributions

**Before committing:**
1. Run `black`, `isort`, and `flake8` to fix formatting issues
2. Ensure MyPy passes without errors
3. Fix any trailing whitespace or other formatting issues
4. Only commit when all pre-commit hooks pass successfully

### Code Quality Requirements

- **MyPy:** All code must pass strict type checking with zero errors
- **Black:** All Python code must be formatted with Black
- **isort:** All imports must be properly sorted and organized
- **flake8:** All code must pass linting checks
- **Tests:** All existing tests must pass before committing

---

## 📚 Documentation Strategy and Policy

**⚠️ IMPORTANT:** This project uses a **Sphinx-first documentation strategy** to prevent documentation sprawl.

### Primary Documentation: Sphinx/RST (docs/source/)

**All user-facing documentation MUST go in Sphinx:**
- API reference (auto-generated from docstrings)
- Configuration guides
- Usage examples
- Installation instructions
- Feature documentation

**Update Sphinx docs when:**
- Adding new features or configuration options
- Changing existing APIs
- Adding examples or tutorials
- Updating installation/setup procedures

### Allowed Markdown Files (Repository Only)

**ONLY these Markdown files are permitted:**
1. `README.md` - GitHub landing page and quick start
2. `CHANGELOG.md` - Version history and release notes
3. `CLAUDE.md` - Development context and instructions (this file)

### Forbidden: Analysis and Temporary Documentation

**❌ DO NOT CREATE:**
- Analysis documents (`*_ANALYSIS.md`, `*_REPORT.md`)
- Implementation plans (`*_IMPLEMENTATION.md`, `*_PLAN.md`)
- Requirements documents (`REQUIREMENTS_*.md`)
- Testing guides outside of Sphinx
- API documentation in Markdown
- Configuration guides in Markdown
- Temporary development notes as permanent files

### Documentation Workflow

1. **New Feature Development:**
   - Write comprehensive docstrings in code
   - Add configuration examples to Sphinx
   - Update API documentation if needed
   - Add to CHANGELOG.md when released

2. **Research/Analysis:**
   - Use temporary files locally (not committed)
   - Document final decisions in code comments or Sphinx
   - Update CLAUDE.md if it affects development workflow

3. **Before Committing:**
   - Check that no new `.md` files are being added (except the 3 allowed)
   - Ensure Sphinx docs are updated if APIs changed
   - Run `make docs` to verify documentation builds

### Building Documentation

```bash
# Build Sphinx documentation
make docs

# Serve documentation locally
make docs-serve
```

### Rationale

This project serves a **commercial QR generation service** where:
- Comprehensive API documentation is essential for development
- Searchable, cross-referenced docs improve productivity
- Auto-generated docs from docstrings stay current
- Professional documentation supports commercial use

**Documentation sprawl previously reached 48 MD + 23 RST files. This policy prevents recurrence.**

---

## 🎯 Test Script Discoverability Policy

**⚠️ IMPORTANT:** All permanent test scripts must be represented in the Makefile for discoverability and consistency.

### Test Script Categories

**✅ MUST be in Makefile:**
- **Regression test scripts** - Cross-platform, visual, compatibility testing
- **Integration test suites** - End-to-end testing, external dependencies
- **Compatibility test scripts** - Multi-version, multi-environment testing
- **Performance benchmarks** - Repeatable performance measurement
- **Build validation scripts** - Distribution verification, deployment checks

**🔄 SHOULD be in Makefile:**
- **Development test utilities** - Frequently used during development
- **Platform-specific tests** - OS-specific or environment-specific testing
- **Documentation tests** - Example validation, documentation consistency

**⚪ MAY be standalone:**
- **One-off test scripts** - Temporary debugging, investigation scripts
- **Experimental scripts** - Proof-of-concept, research testing
- **Personal development tools** - Individual developer utilities

### Implementation Requirements

**Makefile Target Naming:**
- Use descriptive targets: `test-segno-compatibility` not `test-compat`
- Group related tests: `test-cross-platform`, `test-environments`
- Include help text in the `help` target

**Documentation in Makefile:**
```makefile
# Test compatibility across multiple Segno versions
test-segno-compatibility:
	@echo "Testing compatibility with Segno versions 1.3.1 to 1.6.6..."
	$(PYTHON) scripts/test_segno_compatibility.py

# Test distribution build process
test-build:
	@echo "Testing distribution build process..."
	$(PYTHON) -m build --wheel
```

### Benefits of This Policy

1. **Discoverability** - `make help` shows all available tests
2. **Consistency** - Standardized execution patterns
3. **CI/CD Integration** - Easy to include in automated pipelines
4. **Documentation** - Makefile becomes living test documentation
5. **Dependency Management** - Handle prerequisites and setup
6. **Maintenance** - Single place to update test commands

### Enforcement

**During development:**
- When creating new test scripts, add Makefile target immediately
- When reviewing PRs, verify test scripts have corresponding targets
- Use `make help` to audit available vs. actual test scripts

**File naming convention:**
- Permanent tests: `test_*.py` or `*_test.py`
- Temporary tests: `debug_*.py`, `explore_*.py`, `tmp_*.py`

---

## 🧪 QR Generation Testing Framework

[Rest of the content remains the same...]

---

## 🛠️ Development Environment Updates

[Rest of the content remains the same...]

---

## 🔧 Development Context

### Project Architecture
- **Plugin Type:** Segno QR code generation plugin
- **Build System:** Hatchling (modern PEP 517/518 build backend)
- **Configuration:** Comprehensive Pydantic-based system with 14+ models
- **Shape System:** 14 different renderers with factory pattern
- **Processing:** Multi-phase pipeline (Phase 1-4)
- **Output:** Professional SVG with accessibility and interactivity

### Code Conventions
- Use existing Pydantic patterns for new configuration models
- Follow factory pattern for extending shape renderers
- Maintain backward compatibility with existing kwargs API
- Add comprehensive type hints and validation
- Include docstrings and field descriptions

### Testing Requirements
- Run existing test suite: `pytest tests/`
- Add tests for new functionality
- Maintain >90% test coverage
- Test backward compatibility
- Include visual regression tests for shape changes

---

## 📋 GitHub Spec-Kit Integration

**🎯 IMPLEMENTED:** SegnoMMS now includes GitHub Spec-Kit for spec-driven development with AI agents.

### Integration Overview (as of 2025-01-25)

**Spec-Kit Components Installed:**
- ✅ **Specify CLI** - Installed via `uv tool install specify-cli`
- ✅ **Claude Templates** - Generated slash commands for spec-driven workflow
- ✅ **Project Constitution** - Aligned with SegnoMMS development standards
- ✅ **Makefile Targets** - `make spec-*` commands for validation and management
- ✅ **Pre-commit Hooks** - Validation for spec files, constitution, and templates

### Spec-Driven Development Workflow

**Phase 1: Constitution and Principles**
1. **Project Constitution**: `.specify/memory/constitution.md` defines SegnoMMS development principles
2. **Alignment with CLAUDE.md**: Constitution enforces all existing standards (Pydantic v2, Sphinx-first docs, test coverage)
3. **Quality Gates**: Pre-commit hooks validate constitution format and completeness

**Phase 2: Feature Specification**
1. Use `/speckit.constitution` to establish or update project principles
2. Use `/speckit.specify` to create feature specifications from natural language descriptions
3. Use `/speckit.clarify` (optional) for structured clarification questions
4. Specifications are validated for completeness and quality before proceeding

**Phase 3: Technical Planning**
1. Use `/speckit.plan` to generate technical implementation plans
2. Plans must respect SegnoMMS constitution and architecture patterns
3. Use `/speckit.checklist` (optional) to validate plan quality
4. Use `/speckit.analyze` (optional) for cross-artifact consistency

**Phase 4: Implementation**
1. Use `/speckit.tasks` to break down plans into actionable tasks
2. Use `/speckit.implement` to execute tasks with proper validation
3. All implementations must pass existing quality gates (MyPy, tests, pre-commit hooks)
4. Maintain backward compatibility with existing plugin API

### Make Targets for Spec-Kit

**Available Commands:**
- `make spec-check` - Validate spec-kit installation and configuration
- `make spec-validate` - Check existing specification files for syntax and structure
- `make spec-clean` - Clean generated spec files (preserves templates and constitution)
- `make spec-help` - Show complete spec-kit workflow and slash commands

### Integration with Existing Standards

**Documentation Policy:**
- Specifications supplement but do NOT replace Sphinx documentation
- Only the three allowed Markdown files remain: README.md, CHANGELOG.md, CLAUDE.md
- Technical specifications in `specs/` directory are for development workflow only
- All user-facing documentation must still go in `docs/source/` (Sphinx/RST)

**Quality Assurance:**
- All spec-driven features must pass comprehensive test suite
- MyPy type checking, pre-commit hooks, and performance benchmarks apply
- Visual regression testing still required for any shape-related features
- Backward compatibility testing across Segno versions remains mandatory

**Constitutional Compliance:**
- Spec-kit constitution enforces Pydantic v2 + MyPy standards
- Plugin architecture integrity cannot be compromised
- Commercial QR service quality standards apply to all spec-driven features
- Test-driven development remains non-negotiable

### Best Practices for Spec-Kit Usage

**For Feature Development:**
1. **Start with constitution** - Ensure project principles are current and complete
2. **Write clear specifications** - Focus on WHAT users need and WHY, avoid HOW
3. **Validate early** - Use built-in quality checklists before proceeding to implementation
4. **Respect existing architecture** - All technical plans must align with plugin patterns
5. **Test comprehensively** - Maintain >90% test coverage and visual regression validation

**For AI Collaboration:**
1. **Use slash commands sequentially** - Follow the designed workflow phases
2. **Leverage clarification tools** - Better to ask questions than make assumptions
3. **Document decisions** - Include rationale in technical plans and specifications
4. **Validate continuously** - Use `make spec-validate` to catch issues early
5. **Integrate with CI/CD** - Spec changes trigger existing quality gates

### Spec-Kit File Structure

```
.claude/                    # Claude-specific agent configuration
├── commands/              # Slash command definitions
│   ├── speckit.constitution.md
│   ├── speckit.specify.md
│   ├── speckit.plan.md
│   ├── speckit.tasks.md
│   └── speckit.implement.md
└── settings.local.json    # Local agent settings

.specify/                  # Spec-kit templates and memory
├── memory/
│   └── constitution.md    # Project constitution
├── scripts/              # Helper scripts for workflow
│   └── bash/
└── templates/            # Spec-kit templates

specs/                    # Generated specifications (created on first use)
└── [feature-number]-[feature-name]/
    ├── spec.md
    ├── plan.md
    ├── tasks.md
    └── artifacts/
```

**This integration provides structured, AI-enhanced development while preserving all SegnoMMS quality standards.**

---

## 💡 Ideas Backlog (`.specify/ideas/`)

**Purpose**: Long-term feature ideas and refactoring opportunities that aren't ready for formal spec-kit workflows.

### How It Works

The `.specify/ideas/` directory contains markdown files for feature ideas that:
- Are identified during code reviews or development
- Need further investigation before formal specification
- Represent "someday/maybe" improvements
- Track refactoring opportunities

### Workflow

1. **Adding Ideas**: Create a new file `.specify/ideas/[topic].md` with:
   - Problem statement
   - Proposed solution
   - Files affected
   - Success criteria
   - Notes section for ongoing learnings

2. **Promoting to Spec-Kit**: When ready to implement:
   - Use `/speckit.specify` to create a formal spec from the idea
   - Reference the original idea file in the spec
   - Move or archive the idea file after implementation

3. **Maintaining Ideas**: Update idea files as:
   - New information is discovered
   - Related work is completed
   - Priorities change

### Current Ideas

| File | Topic | Priority |
|------|-------|----------|
| `processor-split.md` | Split 2,007-line IntentProcessor monolith | High |
| `pattern-style-config-refactor.md` | Refactor 28-field PatternStyleConfig to Dict pattern | High |
| `test-quality-audit.md` | Audit test coverage and edge cases | Foundation |
| `documentation-improvements.md` | Architecture docs, magic number rationale | Medium |

### Recommended Sequence

1. Test Quality Audit (establish baseline before refactoring)
2. Documentation (preserve knowledge before changes)
3. PatternStyleConfig Refactor (smaller scope)
4. IntentProcessor Split (larger scope)

**Note**: This is an informal extension to spec-kit, not part of the official workflow.

---

## 🧹 Recent Repository Maintenance (2025-01-15)

**Completed:**
- ✅ **Documentation Policy Enforcement** - Removed unauthorized Markdown files (CONTRIBUTING.md, ACTIONS_STATUS.md, PHASE3_COMPLETION_SUMMARY.md)
- ✅ **Dependency Management Cleanup** - Removed duplicate `[project.optional-dependencies]` in favor of `[dependency-groups]`
- ✅ **Generated Artifacts Cleanup** - Removed tracked event JSON files, verified .gitignore patterns
- ✅ **Documentation Updates** - Updated installation docs to use `uv sync` / `make setup` instead of deprecated pip install patterns

**Status:**
- Repository now fully compliant with Sphinx-first documentation policy (only 3 MD files: README.md, CHANGELOG.md, CLAUDE.md)
- Single source of truth for dependencies in `[dependency-groups]`
- All generated artifacts properly gitignored

## 🚧 Development Reminders

- **Multiline Shell Commands Standardization** (In Progress)
  - Most workflows properly externalize scripts to `repo/` directory ✓
  - Some workflows still have inline shell commands (46 instances across 6 workflow files)
  - Complete extraction when workflows are next updated

## Active Technologies
- Python 3.10+ (see pyproject.toml for compatibility matrix) + Segno >=1.5.2, Pydantic >=2.7,<3, typing-extensions >=4.8

## Recent Changes
- Version update: Dropped Python 3.9 (EOL Oct 2025), added Python 3.14, updated to pytest >=9.0.0, black >=25.0.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/systmms) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
