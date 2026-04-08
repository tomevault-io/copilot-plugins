## shock-pulse-analysis

> This document provides comprehensive guidelines for AI agents developing Python packages. Follow these standards to ensure high-quality, maintainable, and professional Python code.

# Python Package Development Guidelines for AI Agents

This document provides comprehensive guidelines for AI agents developing Python packages. Follow these standards to ensure high-quality, maintainable, and professional Python code.

## Package Description Template

**Package Name**: shock-pulse-analysis
**Version**: 0.0.0
**Description**: A Python package for determining time domain features related to shock pulses.  These features are described in MIL-STD-810H Change 1, Method 516.8 Annex A and B.
**Main Purpose**: Identification of the following features from a shock pulse time series:
- measurement system noise floor
- $T_{pre}$: the time interval preceeding the shock pulse
- $T_{e}$: the shock duration
- $T_{E}#: the interval within the shock duration with the highest concentration of energy
- $T_{post}$: the time from the termination of the shock until the measurement signal approaches levels of the measurement system noise floor
**Target Users**: Shock analysts and engineers.

## Implementation Plan

### Architecture Overview

This package follows a simple, focused architecture aligned with its single purpose: identifying time domain features from shock pulse measurements.

#### Package Structure (per Section 3 requirements)

```
src/shock_pulse_analysis/
├── __init__.py              # Exports public functions
├── core/
│   ├── __init__.py
│   ├── noise_floor.py       # detect_noise_floor()
│   ├── irms.py              # compute_irms()
│   ├── crest_factor.py      # compute_crest_factor()
│   └── intervals.py         # find_t_pre(), find_t_e(), find_t_E(), find_t_post()
└── utils/
    ├── __init__.py
    └── plotting.py          # plot_shock_analysis()
```

#### Data Representation

All functions that process shock data accept:
- `signal`: numpy array of measurement values (e.g., acceleration in G)
- `time`: numpy array of corresponding time values in seconds

Alternative: `sample_rate` (float, Hz) can be provided instead of `time` array, from which time array is computed.

#### Core Algorithms (from MIL-STD-810H)

1. **Noise Floor Detection**: Estimate RMS of pre-shock stationary noise
2. **Instantaneous RMS**: `irms(t) = sqrt(a(t)²)` computed point-by-point after DC offset removal
3. **Crest Factor**: `CF = A_pk / RMS` computed in intervals of ~T_e/10
4. **T_E Threshold**: Points where `|a(t)| > A_pk / CF`

#### Algorithm Interpretation Notes

The following interpretations of MIL-STD-810H have been resolved:

1. **T_e Termination - Zero-Crossing Interpretation**
   - The standard states: *"Select zero-crossings for values of t for which a_irms(t) is 20 dB (or preferably 40 dB) below the peak."*
   - Since `a_irms = |a(t)|` is always non-negative, there are no "crossings" in the irms domain
   - **Interpretation**: Use a_irms to identify the region where signal has decayed below threshold, then find the actual zero-crossing of the original signal `a(t)` near that region

2. **T_e Termination - Post-Threshold Behavior**
   - Use the **last time** the signal drops below the -20dB/-40dB threshold
   - **Rationale**: Complex shocks may have multiple rebounds above threshold; we want the final termination point

3. **Instantaneous RMS dB Reference**
   - Reference = 1 grms (per Figure 516.8B-3 note "ref=1grms")
   - Formula: `dB = 20 * log10(a_irms / 1.0)`

4. **Time Interval Representation**
   - Primary representation: absolute time values (start_time, end_time in seconds)
   - Indices and masks are derived on-demand from times + sample_rate
   - **Rationale**: Absolute times are invariant under resampling; indices change with sample_rate

#### Dependencies

- **numpy**: Array operations
- **scipy**: `scipy.signal.detrend()` for DC offset removal
- **matplotlib**: Plotting (optional, for visualization)

### Public API

```python
from shock_pulse_analysis import (
    detect_noise_floor,
    compute_irms,
    compute_crest_factor,
    find_t_pre,
    find_t_e,
    find_t_E,
    find_t_post,
    analyze_shock,        # convenience wrapper
    plot_shock_analysis,  # visualization
)

# Individual functions (explicit dependencies)
noise_rms = detect_noise_floor(signal, time)
irms = compute_irms(signal, time)
t_pre = find_t_pre(signal, time, noise_rms)
t_e = find_t_e(signal, time, irms, noise_rms)
crest_factor = compute_crest_factor(signal, time, t_e)
t_E = find_t_E(signal, time, t_e, crest_factor)
t_post = find_t_post(signal, time, t_e, noise_rms)

# Convenience wrapper (computes everything)
results = analyze_shock(signal, time)

# Results dict contains:
results['noise_floor_rms']   # float: RMS of pre-shock noise (measurement units)
results['t_pre']             # tuple[float, float]: (start_time, end_time) in seconds
results['t_e']               # tuple[float, float]: shock duration interval
results['t_E']               # tuple[float, float]: energy concentration interval
results['t_post']            # tuple[float, float]: post-shock interval
results['crest_factor']      # float: computed CF = A_pk / RMS
results['peak_amplitude']    # float: A_pk (measurement units)
results['peak_time']         # float: time of peak amplitude (seconds)

# Slicing with time array (user already has this from input)
mask = (time >= results['t_e'][0]) & (time <= results['t_e'][1])
shock_segment = signal[mask]

# Visualization
plot_shock_analysis(signal, time, results)
```

### Function Specifications

#### `detect_noise_floor(signal, time)`

**Specification:**
```python
def detect_noise_floor(signal: np.ndarray, time: np.ndarray) -> float:
    """Detect the measurement system noise floor RMS from pre-shock region."""
```

**Returns:** `float` - RMS amplitude of the noise floor (in measurement units, e.g., G)

**MIL-STD-810H Guidance:**
- *"An initial low amplitude stationary random measurement termed the measurement system noise floor."* (§1.2)
- *"The first time interval T_pre is usually well defined and occurs prior to the shock where the measurement represents the measurement system noise floor."* (§1.3)
- The noise floor should be stationary and Gaussian; periodic components indicate signal conditioning issues
- SNR should be ≥60 dB: `20*log10(V_fullscale / V_noisefloor) ≥ 60`

**Implementation Notes:**
- Identify the pre-shock region where signal amplitude is stationary and low
- Compute RMS of this region
- Consider warning if periodic components or non-Gaussian behavior detected

**Dependencies:**
- Depends on: None (first in chain)
- Depended on by: `find_t_pre()`, `find_t_e()`, `find_t_post()`

---

#### `compute_irms(signal, time)`

**Specification:**
```python
def compute_irms(signal: np.ndarray, time: np.ndarray) -> np.ndarray:
    """Compute instantaneous RMS (point-by-point absolute value after DC removal)."""
```

**Returns:** `np.ndarray` - IRMS values, same length as input signal

**MIL-STD-810H Guidance:**
- *"The instantaneous rms level is defined as: irms_a(t) = sqrt(a(t)²) for 0 ≤ t ≤ T"* (§3.2, Annex B)
- *"It is assumed that any DC offset in a digitized measurement signal a(t) has been removed prior to computing irms_a."* (§3.2)
- *"Observe that irms_a is computed point by point. Therefore, A_pk will be the maximum computed irms_a."* (§3.2)
- Reference for dB conversion: 1 grms → `dB = 20 * log10(irms / 1.0)`
- *"It is not recommended that the instantaneous rms values be smoothed through short-time-averaging."* (§3.2)

**Implementation Notes:**
- Remove DC offset using `scipy.signal.detrend(signal, type='constant')`
- Compute `irms = np.abs(detrended_signal)` (since sqrt(x²) = |x|)
- Do NOT smooth or window the result

**Dependencies:**
- Depends on: None
- Depended on by: `find_t_e()`

---

#### `compute_crest_factor(signal, time, t_e)`

**Specification:**
```python
def compute_crest_factor(
    signal: np.ndarray, 
    time: np.ndarray, 
    t_e: tuple[float, float]
) -> float:
    """Compute maximum crest factor over T_e interval."""
```

**Returns:** `float` - Maximum crest factor CF = A_pk / RMS

**MIL-STD-810H Guidance:**
- *"The crest factor CF = A_pk / RMS, associated with the single shock..."* (§1.3.2)
- *"The crest factor is computed in small intervals over the duration T_e (e.g., T_e/10), and the maximum crest factor computed on the individual intervals is defined as CF."* (§1.3.2)
- *"Generally, the larger the CF the greater T_E..."* (§1.3.2)
- *"In the event T_E is not provided, the test operator should assume the CF to be 3."* (default fallback)
- Example from standard: CF ≈ 5 for the example shock

**Implementation Notes:**
- Divide T_e interval into ~10 sub-intervals
- For each sub-interval: compute CF = peak / RMS
- Return the maximum CF across all sub-intervals

**Dependencies:**
- Depends on: `find_t_e()` (needs t_e interval)
- Depended on by: `find_t_E()`

---

#### `find_t_pre(signal, time, noise_rms)`

**Specification:**
```python
def find_t_pre(
    signal: np.ndarray, 
    time: np.ndarray, 
    noise_rms: float
) -> tuple[float, float]:
    """Find the pre-shock interval where signal is at noise floor level."""
```

**Returns:** `tuple[float, float]` - (start_time, end_time) in seconds

**MIL-STD-810H Guidance:**
- *"The first time interval T_pre is usually well defined and occurs prior to the shock where the measurement represents the measurement system noise floor."* (§1.3)
- *"In general, for further processing it is convenient, if possible, to select the interval T_pre of duration equal to T_post and these intervals should be reasonably comparable or equal in length to T_e."* (§1.3)
- Example from standard: T_pre = 0 → 0.617 sec

**Implementation Notes:**
- T_pre ends where the shock begins (first crossing above noise threshold)
- Start is typically time[0] (beginning of recording)
- End aligns with start of T_e

**Dependencies:**
- Depends on: `detect_noise_floor()` (needs noise_rms threshold)
- Depended on by: None (terminal output)

---

#### `find_t_e(signal, time, irms, noise_rms)`

**Specification:**
```python
def find_t_e(
    signal: np.ndarray, 
    time: np.ndarray, 
    irms: np.ndarray, 
    noise_rms: float
) -> tuple[float, float]:
    """Find the shock duration interval T_e."""
```

**Returns:** `tuple[float, float]` - (start_time, end_time) in seconds

**MIL-STD-810H Guidance:**
- *"T_e is termed the shock duration and is defined as the duration from the zero crossing for the first measurement acceleration 'above the instrumentation noise floor' until the perceived 'termination' of the shock."* (§1.3)
- *"For establishing the end of the range of T_e for a simple 'well-behaved,' i.e., sharply decaying shocks, it is recommended that the analyst examine times t at which irms_a(t) is at least 20dB (preferably 40 dB) below irms_a(T_pk), and based upon judgment, select the zero-crossing for defining the end of T_e."* (§3.2, Annex B)
- Example from standard: T_e = 0.617 → 1.56 sec (duration = 0.943 sec)

**Implementation Notes (from Algorithm Interpretation):**
- **Start**: Find first zero-crossing of a(t) where signal rises above noise floor threshold
- **End**: 
  1. Find where irms drops to -20dB (or -40dB) below peak irms
  2. Use the **last time** this threshold is crossed (not first) to handle complex shocks
  3. Find the zero-crossing of original signal a(t) near this region

**Dependencies:**
- Depends on: `compute_irms()`, `detect_noise_floor()`
- Depended on by: `compute_crest_factor()`, `find_t_E()`, `find_t_post()`

---

#### `find_t_E(signal, time, t_e, crest_factor)`

**Specification:**
```python
def find_t_E(
    signal: np.ndarray, 
    time: np.ndarray, 
    t_e: tuple[float, float], 
    crest_factor: float
) -> tuple[float, float]:
    """Find the energy concentration interval T_E within T_e."""
```

**Returns:** `tuple[float, float]` - (start_time, end_time) in seconds

**MIL-STD-810H Guidance:**
- *"T_E represents a 'concentration of energy' duration."* (§1.3.2)
- *"T_E is defined as the minimum length of time that contains any time history magnitudes exceeding in absolute value A_pk / CF."* (§1.3)
- *"T_E is identified by the interval between the first occurrence of A_pk/CF and the last occurrence of A_pk/CF."* (§1.3.3)
- Example from standard: T_E ≈ 0.230 sec (0.625 → 0.860 sec), with A_pk = 98.17 G, CF ≈ 5
- Tolerance for synthesis: 0.8 ≤ T_E/T_E_nominal ≤ 1.2

**Implementation Notes:**
- Compute threshold = A_pk / CF (where A_pk = max(|signal|) within t_e)
- Find first time |signal| exceeds threshold
- Find last time |signal| exceeds threshold
- T_E = (first_time, last_time)

**Dependencies:**
- Depends on: `find_t_e()`, `compute_crest_factor()`
- Depended on by: None (terminal output)

---

#### `find_t_post(signal, time, t_e, noise_rms)`

**Specification:**
```python
def find_t_post(
    signal: np.ndarray, 
    time: np.ndarray, 
    t_e: tuple[float, float], 
    noise_rms: float
) -> tuple[float, float]:
    """Find the post-shock interval from shock termination toward noise floor."""
```

**Returns:** `tuple[float, float]` - (start_time, end_time) in seconds

**MIL-STD-810H Guidance:**
- *"The third time interval T_post is the time from the 'termination' of the shock until the measurement signal approaches or reaches levels of the measurement system noise floor."* (§1.3)
- *"In general, shocks over reasonable characterization/identification times seldom decay to the levels of the pre-shock noise floor."* (§1.3)
- *"Almost assuredly, post-shock instantaneous rms is greater than the pre-shock instantaneous rms."* (§3.2, Annex B)
- *"Generally, criteria for defining and automatically determining T_post are left to the discretion of the analyst."* (§3.2)

**Implementation Notes:**
- Start of T_post = end of T_e
- End of T_post = end of recording, or where signal stabilizes near noise floor
- Post-shock noise floor is typically above pre-shock noise floor

**Dependencies:**
- Depends on: `find_t_e()`, `detect_noise_floor()`
- Depended on by: None (terminal output)

---

#### `analyze_shock(signal, time)`

**Specification:**
```python
def analyze_shock(signal: np.ndarray, time: np.ndarray) -> dict:
    """Convenience wrapper that computes all shock features."""
```

**Returns:** `dict` with keys:
- `'noise_floor_rms'`: float
- `'t_pre'`: tuple[float, float]
- `'t_e'`: tuple[float, float]
- `'t_E'`: tuple[float, float]
- `'t_post'`: tuple[float, float]
- `'crest_factor'`: float
- `'peak_amplitude'`: float
- `'peak_time'`: float

**Implementation Notes:**
- Calls individual functions in sequence
- Handles the dependency chain internally

**Dependencies:**
- Depends on: All individual functions
- Depended on by: None (top-level convenience)

---

#### `plot_shock_analysis(signal, time, results)`

**Specification:**
```python
def plot_shock_analysis(
    signal: np.ndarray, 
    time: np.ndarray, 
    results: dict
) -> matplotlib.figure.Figure:
    """Generate visualization of shock analysis results."""
```

**Returns:** `matplotlib.figure.Figure`

**Implementation Notes:**
- Time history plot with T_pre, T_e, T_E, T_post regions annotated
- IRMS plot (optional subplot)
- Threshold lines (noise floor, A_pk/CF)
- Peak marker

**Dependencies:**
- Depends on: Results from `analyze_shock()` or individual functions
- Depended on by: None (visualization output)

---

### Implementation Phases

#### Phase 1: Project Setup
- Initialize package structure per Section 3
- Configure pyproject.toml, requirements files
- Set up virtual environment
- Initialize git repository

#### Phase 2: Input Validation
- Implement input validation helpers (signal/time arrays, sample_rate)
- Time array generation from sample_rate

#### Phase 3: Noise Floor Detection
- Implement `core/noise_floor.py`
- `detect_noise_floor(signal, time)` → float
- Handle edge cases (short signals, high noise)

#### Phase 4: Instantaneous RMS
- Implement `core/irms.py`
- `compute_irms(signal, time)` → numpy array
- DC offset removal using `scipy.signal.detrend()`

#### Phase 5: Crest Factor
- Implement `core/crest_factor.py`
- `compute_crest_factor(signal, time, t_e)` → float
- Compute CF = A_pk / RMS in intervals of ~T_e/10
- Return maximum CF across intervals

#### Phase 6: Time Interval Identification
- Implement `core/intervals.py`
- `find_t_pre(signal, time, noise_rms)` → tuple
- `find_t_e(signal, time, irms, noise_rms)` → tuple
- `find_t_E(signal, time, t_e, crest_factor)` → tuple
- `find_t_post(signal, time, t_e, noise_rms)` → tuple

#### Phase 7: Convenience Wrapper
- Implement `analyze_shock(signal, time)` → dict
- Calls individual functions in sequence
- Returns complete results dict

#### Phase 8: Plotting Utilities
- Implement `utils/plotting.py`
- `plot_shock_analysis(signal, time, results)`
- Time history with interval annotations
- IRMS visualization

#### Phase 9: Documentation and Examples
- Complete docstrings for all public API
- Create demonstration notebook
- Write README with usage examples

### Out of Scope

The following are explicitly **NOT** part of this package:
- Signal filtering (bandpass, lowpass, etc.)
- Resampling or interpolation
- Shock Response Spectrum (SRS) computation
- FFT or frequency domain analysis
- Signal synthesis or generation
- File I/O for specific data formats

These capabilities belong in separate, specialized packages.

## Critical Reference - MANDATORY READING

The following document describes the features to be identified by this package in detail and suggests criteria and methods for their determination:

[MIL-STD-810H Change 1, Method 516.8 Annex A and B](mil-std-810H-change1-method-516-8-shock-annex-a-b.pdf)

**Extracted Text Reference**: For convenience, the complete text content from the PDF has been extracted and is available at [mil-std-810H-change1-method-516-8-shock-annex-a-b.txt](mil-std-810H-change1-method-516-8-shock-annex-a-b.txt)

## Required Standards and Practices

### 1. Python Coding Standards Compliance

All code must comply with the following standards:

- **PEP 8** - Style Guide for Python Code
- **PEP 257** - Docstring Conventions
- **PEP 484** - Type Hints (Python 3.5+)
- **PEP 20** - The Zen of Python principles

**Tools for Compliance**:
- Use `black` for code formatting
- Use `flake8` for linting
- Use `mypy` for static type checking
- Use `isort` for import sorting

### 2. Test-Driven Development (TDD)

Follow TDD methodology:

1. **Red** - Write a failing test first
2. **Green** - Write minimal code to pass the test
3. **Refactor** - Improve code while keeping tests passing

**Testing Requirements**:
- Minimum 90% code coverage
- All public methods must have tests
- Edge cases and error conditions must be tested
- Tests must be deterministic and independent

### 3. Package Structure

Use the following standard package structure:

```
[package_name]/
├── README.md
├── setup.py
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
├── .gitignore
├── .env.example
├── src/
│   └── [package_name]/
│       ├── __init__.py
│       ├── main.py
│       ├── core/
│       │   ├── __init__.py
│       │   └── [module].py
│       └── utils/
│           ├── __init__.py
│           └── [utility_module].py
├── tests/
│   ├── __init__.py
│   ├── test_main.py
│   ├── core/
│   │   ├── __init__.py
│   │   └── test_[module].py
│   └── utils/
│       ├── __init__.py
│       └── test_[utility_module].py
├── docs/
│   ├── README.md
│   └── [documentation_files].md
├── examples/
│   ├── demo.ipynb
│   └── [example_scripts].py
└── notebooks/
    └── package_demonstration.ipynb
```

### 4. Test Organization

Each Python module must have a corresponding test file:

- For `src/package_name/module.py` → `tests/test_module.py`
- For `src/package_name/core/feature.py` → `tests/core/test_feature.py`
- Mirror the source directory structure in the tests directory
- One test file per module - consolidate all tests for a module in its single test file

### 5. Unit Testing with unittest

Use Python's built-in `unittest` framework:

```python
import unittest
from unittest.mock import Mock, patch, MagicMock

class TestModuleName(unittest.TestCase):
    
    def setUp(self):
        """Set up test fixtures before each test method."""
        pass
    
    def tearDown(self):
        """Clean up after each test method."""
        pass
    
    def test_function_name_expected_behavior(self):
        """Test description following naming convention."""
        # Arrange
        # Act
        # Assert
        pass
    
    @patch('module.dependency')
    def test_function_with_mock(self, mock_dependency):
        """Test with mocked dependencies."""
        pass

if __name__ == '__main__':
    unittest.main()
```

**Testing Commands**:
```bash
# Run all tests
python -m unittest discover tests/

# Run specific test file
python -m unittest tests.test_module

# Run with coverage
python -m coverage run -m unittest discover tests/
python -m coverage report
python -m coverage html
```

### 6. Virtual Environment Setup

Always use virtual environments:

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
# venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Deactivate when done
deactivate
```

**Environment Files**:
- `requirements.txt` - Production dependencies
- `requirements-dev.txt` - Development dependencies (testing, linting, etc.)
- `.env.example` - Template for environment variables

### 7. Git Ignore Configuration

Use this comprehensive `.gitignore`:

```gitignore
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# PyInstaller
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

# Translations
*.mo
*.pot

# Django stuff:
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# Flask stuff:
instance/
.webassets-cache

# Scrapy stuff:
.scrapy

# Sphinx documentation
docs/_build/

# PyBuilder
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
.python-version

# pipenv
Pipfile.lock

# PEP 582
__pypackages__/

# Celery stuff
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/

# IDEs
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Custom
data/
logs/
temp/
```

### 8. Jupyter Notebook for Demonstration

Create `notebooks/package_demonstration.ipynb` that includes:

1. **Installation instructions**
2. **Import statements and setup**
3. **Basic usage examples**
4. **Advanced feature demonstrations**
5. **Real-world use cases**
6. **Performance benchmarks (if applicable)**
7. **Troubleshooting common issues**

**CRITICAL: Kernel Restart Requirements**:
- **Always restart the notebook kernel after modifying package source code**
- Use: `Kernel → Restart` or `Kernel → Restart & Clear Output` in Jupyter/VS Code
- Python imports are cached - kernel restart is required to reload changed modules
- Add this guidance prominently in notebook cells when demonstrating development workflow
- Test notebooks must be re-executed from top after any source code changes

**Notebook Structure Template**:
```markdown
# [Package Name] Demonstration

## 1. Installation
## 2. Basic Usage
## 3. Core Features
## 4. Advanced Examples
## 5. Integration Examples
## 6. Performance Analysis
## 7. Troubleshooting
```

### 9. Git Usage Requirements

Follow these Git practices:

- Initialize repository: `git init`
- Use meaningful commit messages following [Conventional Commits](https://www.conventionalcommits.org/)
- Create feature branches: `git checkout -b feature/feature-name`
- Use pull requests for code review
- Tag releases: `git tag -a v1.0.0 -m "Version 1.0.0"`

**Commit Message Format**:
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

#### 9.1 Atomic Commits

Make frequent, small commits that each represent a single logical change:

- **One commit per logical change** - don't bundle unrelated changes
- **Tests should pass after each commit** - never commit broken code
- **Commit after each TDD cycle** - Red → Green → Refactor can be 1-3 commits
- **Commit message should fully describe the change** - no need to read the diff

**Good atomic commit examples**:
```
test: Add failing test for noise_floor detection with empty signal
feat: Implement noise_floor detection for empty signal case
refactor: Extract threshold calculation to helper function
```

**Bad commit examples**:
```
WIP
Fixed stuff
Updates to multiple files
```

#### 9.2 Branch-Based Development Workflow

Use feature branches for each development phase:

**Branch Naming Convention**:
```
feature/phase-N-description   # New features (e.g., feature/phase-2-noise-floor)
fix/issue-description         # Bug fixes
docs/description              # Documentation only
refactor/description          # Code refactoring
```

**Workflow**:
```bash
# Start new phase
git checkout main
git checkout -b feature/phase-2-noise-floor

# ... develop with frequent atomic commits ...

# When phase is complete (tests pass, linting clean)
git checkout main
git merge feature/phase-2-noise-floor
git branch -d feature/phase-2-noise-floor
```

**Branch Merge Criteria**:
- [ ] All tests pass
- [ ] Code formatting clean (`black --check`)
- [ ] Linting clean (`flake8`)
- [ ] Phase objectives met
- [ ] Documentation updated if needed

**Benefits**:
- `main` branch always in working state
- Easy to abandon/reset incomplete work
- Clear checkpoint when phase is complete
- Supports future collaboration

## Additional Recommended Practices

### Configuration Files

Include these configuration files:

**`pyproject.toml`**:
```toml
[build-system]
requires = ["setuptools>=45", "wheel", "setuptools_scm"]
build-backend = "setuptools.build_meta"

[project]
name = "[package_name]"
version = "[version]"
description = "[description]"
authors = [{name = "[author]", email = "[email]"}]
license = {text = "MIT"}
readme = "README.md"
requires-python = ">=3.8"
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.8",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
]

[tool.black]
line-length = 88
target-version = ['py38']

[tool.isort]
profile = "black"

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
```

### Documentation Standards

Include comprehensive documentation:

- **README.md** with installation, usage, and examples
- **LICENSE** file


## Checklist for Agents

Before considering a Python package complete, ensure:

- [ ] All code follows PEP 8 standards
- [ ] TDD methodology was followed
- [ ] Test coverage is ≥90%
- [ ] All modules have corresponding test files
- [ ] Virtual environment is set up and documented
- [ ] .gitignore file is comprehensive
- [ ] Demonstration Jupyter notebook is complete and all cells run without errors
- [ ] Git repository is properly initialized
- [ ] Package structure follows standards
- [ ] Documentation is comprehensive
- [ ] Type hints are used throughout
- [ ] Error handling is robust
- [ ] Performance is acceptable
- [ ] Security considerations are addressed

Before passing control back the user following any code change, ALWAYS:
- [ ] Restart the Jupyter notebook kernel and re-execute all cells to ensure changes are reflected
- [ ] Run all tests
- [ ] Check code formatting with `black` and linting with `flake8` and ensure no issues
- [ ] Recommend whether an atomic commit should be made

## Resources

- [PEP 8 Style Guide](https://www.python.org/dev/peps/pep-0008/)
- [PEP 257 Docstring Conventions](https://www.python.org/dev/peps/pep-0257/)
- [Python Packaging Guide](https://packaging.python.org/)
- [unittest Documentation](https://docs.python.org/3/library/unittest.html)
- [Test-Driven Development Guide](https://testdriven.io/test-driven-development/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/natecostello)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/natecostello)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
