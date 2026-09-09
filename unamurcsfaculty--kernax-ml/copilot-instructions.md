## kernax-ml

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Kernax is a JAX-based kernel library for Gaussian Processes, implementing various covariance functions with automatic differentiation and JIT compilation support. The library is built on Equinox and follows a single-class abstract-final pattern, with per-hyperparameter customizable parametrisation.

## Writing Style: No Process Narration

Treat this codebase — including `.md` design/planning docs, docstrings, and code comments —
as final production content, not a record of how we got here.

- Never write comments or doc prose that reference the history of the discussion: no "unlike
  what I wrote before", "contrary to the earlier plan", "corrected from v1", "we used to do X
  but now Y", changelog-style narration inside a design doc, etc. If something changed during
  drafting, just present the current, correct version — don't narrate the correction.
- Don't use code comments or doc text to show your work or prove you understood a correction.
  That belongs in the conversation turn, not in a file a future maintainer will read.
- Explain *why* only when it encodes a non-obvious constraint that will still matter to a
  future reader (a hidden invariant, a real tradeoff, a workaround for an external bug) — not
  "why we don't do it the other way we once considered."
- This applies to design/planning `.md` files too: write them as the current state of the
  plan, not as a diff against previous drafts of themselves.

## Architecture

### Single-Class Pattern

Each kernel is a single class extending `AbstractKernel` (or a subclass like `AbstractStationaryKernel`):

- Stores hyperparameters in **wrapped form** via private fields (e.g., `_length_scale`) with `@property` accessors that unwrap them
- Each hyperparameter has its own **`AbstractParametrisation`** (default: `LogExpParametrisation`) controlling its transformation during optimization
- Implements `pairwise(self, x1, x2)` as an instance method decorated with `@filter_jit`
- Implements `replace(**kwargs)` for immutable parameter updates via `eqx.tree_at()`
- Holds `engine` as a static field (default: `DenseEngine`) controlling matrix construction

### AbstractKernel Base Class

`AbstractKernel` (kernax/AbstractKernel.py) extends `AbstractModule` (which extends `eqx.Module`) and provides:

- **Abstract interface**: Declares `pairwise(self, x1, x2)` and `replace(self, ...)` as abstract methods
- **`__call__`**: Delegates to `self.engine.__call__()`, which handles dimension detection, vectorization, and NaN handling
- **Operator overloading**: `+`, `*`, `-` operators create `SumModule`, `ProductModule`, `NegModule` respectively (defined in `AbstractModule`)

Subclass hierarchy:
- `AbstractStationaryKernel`: adds `distance_function` static field
- `AbstractDotProductKernel`: for dot-product-based kernels

### Kernel Categories

1. **Base Kernels** (implement `pairwise` as instance method):
   - SE (Squared Exponential, aka RBF or Gaussian)
   - Linear, Affine, Polynomial, Sigmoid
   - Matern (1/2, 3/2, 5/2)
   - Periodic, Rational Quadratic
   - Constant, Variance, Feature, WhiteNoise

2. **Operator Modules** (kernax/operators/): Combine two modules
   - `SumModule`: Adds outputs of two modules
   - `ProductModule`: Multiplies outputs of two modules

3. **Wrapper Modules** (kernax/wrappers/): Transform or modify kernel behavior
   - `ExpModule`: Applies exponential
   - `LogModule`: Applies logarithm
   - `NegModule`: Negates output
   - `BatchModule`: Adds batch handling with distinct hyperparameters per batch element
   - `BlockKernel`: Constructs block covariance matrices for grouped data
   - `BlockDiagKernel`: Block-diagonal covariance matrices, specialized version of BlockKernel
   - `ActiveDimsModule`: Selects specific input dimensions before kernel computation
   - `ARDKernel`: Applies Automatic Relevance Determination (different length scale per dimension)

4. **Computation Engines** (kernax/engines.py): Control how covariance matrices are computed
   - `DenseEngine` (default): Computes full covariance matrices
   - `SafeDiagonalEngine`: Returns diagonal matrices (uses conditional check for input equality)
   - `FastDiagonalEngine`: Returns diagonal matrices (assumes x1 == x2, faster but requires constraint)
   - `SafeRegularGridEngine`: Exploits regular grid structure with runtime checks
   - `FastRegularGridEngine`: Exploits regular grid structure without checks (faster but requires constraint)
   - All kernels accept an `engine` parameter for specialized computation patterns

## Development Commands

### Running Python Code
```bash
# Navigate to the kernax directory
cd kernax

# Run Python scripts that import kernax-ml
python3 script.py
```

### Testing Kernels
```bash
# Import and test a kernel in Python REPL
cd kernax
python3
>>> from kernax-ml import SEKernel
>>> import jax.numpy as jnp
>>> kernel = SEKernel(length_scale=1.0)
>>> kernel(jnp.array([1.0]), jnp.array([2.0]))
```

### Running Tests
```bash
# Run all tests
make test

# Run tests with coverage report
make test-cov

# Run tests and generate Allure HTML report
make test-allure

# Run linters (ruff, mypy)
make lint

# Format code with tabs
make format
```

All test outputs (htmlcov, allure-results, allure-report) are saved in `tests/out/` directory.

## Version Management

### Upgrading to a New Version

When the user asks to "upgrade the project to version X.Y.Z" or "prepare for release vX.Y.Z-alpha", follow this procedure:

1. **Review Git History**
   ```bash
   # Check commits since last version tag
   git log --oneline <last-version-tag>..HEAD

   # Get detailed stats for each commit
   git show <commit-hash> --stat
   ```

2. **Update Version Numbers** (in order):
   - `pyproject.toml`: Update `version = "vX.Y.Z-alpha"` (line 7, with "v" prefix)
   - `kernax/__init__.py`: Update `__version__ = "X.Y.Z-alpha"` (line 9, without "v" prefix)

3. **Update CHANGELOG.md**:
   - Add new version section under `## [Unreleased]` with format: `## [X.Y.Z-alpha] - YYYY-MM-DD`
   - Group changes by category (use git commit messages as guide):
     - **Added**: New features, kernels, or capabilities
     - **Changed**: Modifications to existing functionality
     - **Fixed**: Bug fixes and corrections
     - **Enhanced**: Performance improvements or optimizations
     - **Removed**: Deprecated features removed
     - **Technical Details**: Implementation notes for developers
   - Use descriptive bullet points with context (not just "fixed bug X")
   - Reference affected files/classes when relevant
   - Include test coverage information for new features
   - Add comparison link at bottom: `[X.Y.Z-alpha]: https://github.com/SimLej18/kernax-ml/compare/vPREV...vX.Y.Z-alpha`
   - Update `[Unreleased]` link to point to new version

4. **Changelog Writing Guidelines**:
   - Start with user-facing impact, then technical details
   - Use present tense for descriptions ("Adds", "Fixes", "Changes")
   - Include code examples for new APIs when helpful
   - Mention breaking changes prominently at the top of relevant sections
   - Group related changes together (e.g., all printing improvements in one bullet)
   - Cross-reference related changes across sections when applicable

5. **Version Number Format**:
   - Alpha releases: `vX.Y.Z-alpha` (development/testing)
   - Beta releases: `vX.Y.Z-beta` (feature complete, testing)
   - Release candidates: `vX.Y.Z-rc.N` (final testing)
   - Stable releases: `vX.Y.Z` (production ready)
   - Follow semantic versioning:
     - X (major): Breaking changes
     - Y (minor): New features, backward compatible
     - Z (patch): Bug fixes, backward compatible

6. **Pre-Release Checklist**:
   - Ensure all tests pass (`make test`)
   - Verify linting passes (`make lint`)
   - Check test coverage remains above 90%
   - Review CHANGELOG for completeness and accuracy
   - Confirm all new features are documented
   - Verify version numbers are consistent across files

7. **Do NOT automatically**:
   - Create git tags (wait for user confirmation)
   - Push to remote repository
   - Build/publish packages to PyPI
   - Update documentation sites

### Version History Reference

Check `CHANGELOG.md` for full version history. Recent versions:
- **v0.5.5-alpha** (2026-04-20): Architectural rewrite — single-class abstract-final pattern, per-HP parametrisation, mean functions
- **v0.4.4-alpha** (2026-02-06): Kernel modification API, VarianceKernel, printing improvements
- **v0.4.3-alpha** (2026-02-05): Initial kernel modification support
- **v0.4.2-alpha** (2026-02-03): FeatureKernel, BlockKernel API refactoring
- **v0.4.1-alpha** (2026-02-02): Computation engine fixes, DiagKernel removal
- **v0.4.0-alpha** (2025-01-31): Parameter transform system

## Implementation Guidelines

### Adding a New Kernel

1. Create a class inheriting from `AbstractKernel` (or `AbstractStationaryKernel` for distance-based kernels)
2. Declare fields: `engine` (static), `_param_parametrisation`, `_param` (with `converter=jnp.asarray`)
3. Add a `@property` to unwrap each hyperparameter via its parametrisation
4. Implement `__init__`: validate inputs, wrap hyperparameters via `parametrisation.wrap()`
5. Implement `pairwise(self, x1, x2)` with `@filter_jit`
6. Implement `replace(**kwargs)` using `eqx.tree_at()` on the wrapped field
7. Add the class to `__init__.py` imports and `__all__`

Example:
```python
from __future__ import annotations
import equinox as eqx
from equinox import filter_jit
from jax import Array
import jax.numpy as jnp
from .AbstractKernel import AbstractKernel
from .engines import AbstractEngine, DenseEngine
from .parametrisations import AbstractParametrisation, LogExpParametrisation


class MyKernel(AbstractKernel):
    engine: AbstractEngine = eqx.field(static=True)
    _my_param_parametrisation: AbstractParametrisation = eqx.field()
    _my_param: Array = eqx.field(converter=jnp.asarray)

    @property
    def my_param(self) -> Array:
        return self._my_param_parametrisation.unwrap(self._my_param)

    def __init__(self,
                 my_param: float | Array,
                 my_param_parametrisation: AbstractParametrisation = LogExpParametrisation(),
                 engine: AbstractEngine = DenseEngine):
        my_param = jnp.asarray(my_param)
        if jnp.any(my_param <= 0):
            raise ValueError("`my_param` must be positive.")
        self._my_param_parametrisation = my_param_parametrisation
        self._my_param = self._my_param_parametrisation.wrap(my_param)
        self.engine = engine

    @filter_jit
    def pairwise(self, x1: Array, x2: Array) -> Array:
        # Implement kernel computation
        return ...

    def replace(self, my_param: None | float | Array = None, **kwargs) -> MyKernel:
        if my_param is None:
            return self
        my_param = jnp.asarray(my_param)
        return eqx.tree_at(
            lambda k: k._my_param,
            self,
            self._my_param_parametrisation.wrap(my_param)
        )
```

### Import Patterns

- Within kernax modules: Use relative imports `from .AbstractKernel import` (preferred for avoiding circular imports)
- External imports: Use absolute imports `from kernax import` for clarity
- All imports have been standardized to use relative imports within the kernax package to fix mypy type checking issues

### JAX and Equinox Considerations

- All kernel computations must use `jax.numpy` instead of `numpy`
- Use `@filter_jit` from Equinox for JIT compilation (handles PyTrees correctly)
- Hyperparameters are automatically converted to JAX arrays via `eqx.field(converter=jnp.asarray)`
- PyTree registration is automatic through `eqx.Module` inheritance
- Use `eqx.field(static=True)` for non-differentiable parameters (e.g., dimensions, boolean flags)
- Equinox provides clean separation between differentiable and static fields

### Testing Guidelines

The test suite uses pytest with Allure reporting and achieves 94% code coverage. Tests are organized as:

- **test_base_kernels.py**: Tests for all base kernel implementations (SE, Linear, Matern, Periodic, etc.)
  - Mathematical properties (symmetry, positive semi-definiteness)
  - Dimension handling (scalar, vector, matrix)
  - NaN handling for missing data
  - Hyperparameter variations
  - String representations

- **test_kernel_compositions.py**: Tests for kernel composition operations
  - Operator overloading (+, -, *, unary -)
  - Explicit constructor tests (SumKernel, ProductKernel)
  - Scalar auto-conversion to ConstantKernel
  - Wrapper kernels (ExpKernel, LogKernel, NegKernel)
  - Complex compositions and mathematical properties (associativity, distributivity)

- **test_wrapper_kernels.py**: Tests for wrapper kernel implementations
  - BatchKernel, BlockKernel, BlockDiagKernel
  - ActiveDimsKernel, ARDKernel
  - Batch handling and dimension selection
  - Block structure verification

When adding new kernels or features:
1. Add comprehensive tests covering all use cases
2. Use `@allure.title` and `@allure.description` decorators
3. Use pytest parametrization for multiple scenarios
4. Test string representations with `test_str_representation()`
5. Verify mathematical properties when applicable
6. Ensure coverage remains above 90%

### Code Quality

- **Linting**: Use `make lint` to run ruff and mypy
  - Code follows ruff formatting with tabs (line length 100)
  - Type hints use relative imports and `from __future__ import annotations`
  - JAX operations may need `# type: ignore` comments for mypy compatibility

- **Formatting**: Use `make format` to auto-format code
  - Indent style: tabs
  - Quote style: double quotes
  - Line ending: auto

---
> Source: [UNamurCSFaculty/kernax-ml](https://github.com/UNamurCSFaculty/kernax-ml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
