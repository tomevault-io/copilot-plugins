## viral-ngs

> This document provides guidance for AI assistants (Claude Code, GitHub Copilot, etc.) working on this repository.

# AGENTS.md

This document provides guidance for AI assistants (Claude Code, GitHub Copilot, etc.) working on this repository.

## Overview

viral-ngs is a consolidated monorepo for viral NGS (Next-Generation Sequencing) analysis tools. It provides:

- **Core utilities**: Read manipulation, Illumina demultiplexing, file handling, QC
- **Assembly**: Genome assembly, scaffolding, gap filling
- **Classification**: Metagenomic classification, taxonomy filtering, k-mer analysis
- **Phylogenetics**: Variant calling, consensus generation, annotation

**Related resources:**
- Command-line documentation: https://viral-ngs.readthedocs.org/
- Higher-level pipelines: https://github.com/broadinstitute/viral-pipelines

---

## Development Environment

### Docker-Centric Development

Development is **intentionally docker-centric**. Developers need:
- Docker
- Git
- Text/code editor

### Development Workflow

1. Clone the repository:
   ```bash
   git clone https://github.com/broadinstitute/viral-ngs.git
   ```

2. Run the container with local checkout mounted:
   ```bash
   docker run -it --rm \
     -v $(pwd):/opt/viral-ngs/source \
     quay.io/broadinstitute/viral-ngs:main-core
   ```

3. If modifying conda dependencies, install them inside the container:
   ```bash
   micromamba install <packages>
   ```

4. Test code interactively:
   ```bash
   cd /opt/viral-ngs/source
   pytest -rsxX -n auto tests/unit
   ```

5. Push changes to GitHub for automated CI testing

### Running Tests

```bash
# Run all unit tests in the core image
docker run --rm \
  -v $(pwd):/opt/viral-ngs/source \
  quay.io/broadinstitute/viral-ngs:main-core \
  pytest -rsxX -n auto /opt/viral-ngs/source/tests/unit

# Run specific module tests
docker run --rm \
  -v $(pwd):/opt/viral-ngs/source \
  quay.io/broadinstitute/viral-ngs:main-classify \
  pytest -rsxX -n auto /opt/viral-ngs/source/tests/unit/classify
```

**Important: Testing source code changes requires re-installing the package.**
The `-v` mount makes your local files visible on disk, but `viral_ngs` is already installed as a package inside the container image. Python imports resolve to the *installed* copy, not your mounted source files. If you've modified files under `src/viral_ngs/`, you must re-install before running tests:

```bash
# Run tests with local source changes applied
docker run --rm \
  -v $(pwd):/opt/viral-ngs/source \
  quay.io/broadinstitute/viral-ngs:main-core \
  bash -c "pip install -e /opt/viral-ngs/source --quiet && pytest -rsxX -n auto /opt/viral-ngs/source/tests/unit"
```

Changes to test files (`tests/`) and test inputs (`tests/input/`) are picked up automatically via the volume mount — the re-install is only needed when modifying the `src/viral_ngs/` package code.

Running pytest directly on the host will generally not work — most dependencies (bioinformatics tools, conda packages) are only available inside the Docker containers. Always test inside Docker.

**Test conventions:**
- Uses pytest (not nose or unittest)
- Test files in `tests/unit/<module>/`
- Test input files in `tests/input/<TestClassName>/`
- Access via `viral_ngs.core.file.get_test_input_path(self)`
- Custom marker: `@pytest.mark.slow` for slow tests

---

## Code Architecture

### Directory Structure

```
viral-ngs/
├── pyproject.toml              # Package configuration
├── src/viral_ngs/
│   ├── __init__.py             # Version detection
│   ├── py.typed                # PEP 561 marker
│   │
│   ├── # Command modules (CLI entry points)
│   ├── illumina.py             # Illumina demux commands
│   ├── read_utils.py           # Read manipulation commands
│   ├── assembly.py             # Assembly commands
│   ├── metagenomics.py         # Classification commands
│   ├── interhost.py            # Phylo commands
│   │
│   ├── core/                   # Core library (shared utilities + tool wrappers)
│   │   ├── __init__.py         # Tool/InstallMethod classes
│   │   ├── samtools.py         # Tool wrapper
│   │   ├── picard.py           # Tool wrapper
│   │   ├── file.py             # File utilities
│   │   ├── misc.py             # General utilities
│   │   └── ...
│   │
│   ├── assemble/               # Assembly tool wrappers
│   │   ├── __init__.py
│   │   ├── spades.py
│   │   └── ...
│   │
│   ├── classify/               # Classification tool wrappers
│   │   ├── __init__.py
│   │   ├── kraken2.py
│   │   └── ...
│   │
│   └── phylo/                  # Phylogenetics tool wrappers
│       ├── __init__.py
│       ├── mafft.py
│       └── ...
│
├── docker/
│   ├── Dockerfile.baseimage    # Base with conda/python
│   ├── Dockerfile.core         # Core tools
│   ├── Dockerfile.assemble     # + assembly tools
│   ├── Dockerfile.classify     # + classification tools
│   ├── Dockerfile.phylo        # + phylo tools
│   ├── Dockerfile.mega         # All tools combined
│   ├── install-conda-deps.sh
│   └── requirements/
│       ├── baseimage.txt
│       ├── core.txt
│       ├── core-x86.txt        # x86-only core packages
│       ├── assemble.txt
│       ├── assemble-x86.txt    # x86-only assembly packages
│       ├── classify.txt
│       ├── classify-x86.txt    # x86-only classify packages
│       ├── phylo.txt
│       └── phylo-x86.txt       # x86-only phylo packages
│
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   ├── core/
│   │   ├── assemble/
│   │   ├── classify/
│   │   └── phylo/
│   └── input/
│
├── scripts/                    # Utility scripts
├── .github/workflows/
│   └── docker.yml              # CI/CD workflow
└── docs/
```

### Command Module Pattern

Command modules define CLI entry points:

```python
__commands__ = []

def parser_<command_name>(parser=argparse.ArgumentParser()):
    # Define arguments
    return parser

def main_<command_name>(args):
    # Implementation
    pass

__commands__.append(('command_name', parser_command_name))
```

### Tool Wrapper Pattern

Tool wrappers in `core/`, `assemble/`, `classify/`, `phylo/`:

```python
import viral_ngs.core as core

class SamtoolsTool(core.Tool):
    def __init__(self, install_methods=None):
        if install_methods is None:
            install_methods = [core.PrexistingUnixCommand('samtools')]
        super().__init__(install_methods=install_methods)

    def execute(self, command, *args):
        # Run samtools with arguments
        pass
```

### New Tool Integration Checklist

When adding a new bioinformatics tool wrapper to viral-ngs, follow these guidelines to ensure compatibility with downstream workflows and reduce review iteration.

**Design principle**: Downstream pipeline languages (WDL, Nextflow, Snakemake) and end users invoke our exposed CLI commands -- they should never need to drop into inline Python to get things done. This is why API completeness and pipeline interoperability matter: every useful knob should be reachable from the command line, and every tool's outputs should be consumable by other tools' inputs through CLI commands alone.

#### API Completeness

Expose the real, useful parameters of the underlying tool -- sensitivity/specificity thresholds, resource controls (`--splits`, `--threads`), mode switches -- with sensible defaults. A wrapper that only exposes `num_threads` is incomplete.

Pipeline authors using WDL or other workflow languages can only access what's exposed on the CLI; if a threshold is hardcoded in Python, they can't tune it without forking. If values are intentionally hardcoded, document the rationale.

**Example**: A metagenomics classifier should expose minimum score thresholds, confidence levels, and filtering presets -- not just thread count.

#### Pipeline Interoperability

Document or implement how the new tool's outputs feed into existing viral-ngs tools and WDL pipelines through CLI commands. If outputs use a custom format, consider adding a converter to a standard format (e.g., Kraken2-style TSV for `filter_bam_to_taxa` and Krona).

If CLI commands are meant to chain, verify they actually connect end-to-end with no undocumented intermediate steps or manual transformations. A pipeline author should be able to wire tool A's output into tool B's input using only exposed CLI commands.

**Example**: If a classifier produces per-read classifications, either make the output compatible with `filter_bam_to_taxa` (Kraken2 format), or provide a converter command like `classifier_to_kraken2`.

#### Empty-Input Output Contracts

Tool wrappers must handle empty input gracefully and produce the *semantically correct* empty output for that file type. What "empty" means depends on the format:

- **TSV**: Header-only file (column names, zero data rows)
- **BAM**: Header-only BAM (valid BAM structure with headers but no reads)
- **Directory**: Empty directory with the expected structure
- **FASTA**: Zero-byte file or file with no sequences

Check the upstream tool's documentation to determine the correct representation. Never crash on valid empty input. Test this explicitly.

**Example**: If a metagenomics classifier receives an empty BAM, it should produce an empty classification TSV (header only), not raise an exception or write a malformed file.

#### Custom Format Documentation

Non-standard file formats must document:

1. The exact column layout / schema
2. What upstream tool or script produces them
3. Include validation of the expected structure at parse time

Don't silently accept malformed input.

**Example**: If your tool expects "augmented PAF" (standard minimap2 PAF + two trailing numeric columns), document what produces this format and validate that the trailing fields are actually numeric.

#### Test Patterns

Include tests for:

1. **Empty/edge-case inputs** -- See output contracts above. Verify graceful handling and correct empty output format.
2. **CLI parser round-trip** -- Test through argparse (`parser_command().parse_args([...])`), not just calling the `main_` function directly. This ensures CLI help text and argument parsing work correctly.
3. **Pipeline integration** -- Chain command A's output into command B and verify the result. For example, run the classifier on a BAM, then feed its output to a downstream filter and verify it works end-to-end.

#### Dockerfile Import Verification (Belt-and-Suspenders)

When adding a new module that brings in new compiled dependencies (C extensions, CUDA, etc.), consider adding it to the Dockerfile's `RUN python -c "from viral_ngs.X import ..."` verification line. This catches broken dependency installs at build time, especially on ARM64.

**Example**: Adding `virnucpro` to `classify/__init__.py` with a new `duckdb` dependency → update `Dockerfile.classify` to include `virnucpro` in the classify tools import check.

---

## Import Patterns

### Standard imports

```python
# Within command modules (illumina.py, assembly.py, etc.)
import viral_ngs.core as core
import viral_ngs.core.file as util_file
import viral_ngs.core.misc as util_misc

# Using tools
samtools = core.samtools.SamtoolsTool()
bwa = core.bwa.BwaTool()

# Using utilities
util_file.mkstempfname()
util_misc.available_cpu_count()
```

### Within core/ modules (use relative imports)

```python
from . import samtools, picard
from .file import mkstempfname
from .misc import available_cpu_count
```

### Within subpackages (assemble/, classify/, phylo/)

```python
import viral_ngs.core as core
import viral_ngs.core.file as util_file

# For other tools in same subpackage
from . import mummer, mafft
```

### Key rules

1. **Prefer full imports**: `import viral_ngs.core.samtools` over `from viral_ngs.core import samtools`
2. **Use relative imports within packages**: `from . import X` inside core/, assemble/, etc.
3. **No backward compat stubs**: `viral_ngs.tools` and `viral_ngs.util` don't exist

---

## Dependencies

### Conda-First Approach

ALL runtime dependencies are installed via conda for speed and binary compatibility.
The `pyproject.toml` has empty dependencies - conda handles everything.

### Adding Dependencies

1. Check conda availability:
   ```bash
   micromamba search <package>              # default channel
   micromamba search -c bioconda <package>  # bioconda channel
   ```

2. Add to appropriate requirements file:
   - `docker/requirements/core.txt` - core dependencies
   - `docker/requirements/assemble.txt` - assembly-specific
   - `docker/requirements/classify.txt` - classification-specific
   - `docker/requirements/phylo.txt` - phylo-specific

3. For x86-only packages (no ARM64 build), add to the appropriate `-x86.txt` file:
   - `core-x86.txt` - novoalign, mvicuna
   - `classify-x86.txt` - bmtagger, kallisto
   - `phylo-x86.txt` - table2asn

### Dependency Resolution

When building derivative images, ALL dependencies (including x86-only) must be installed in a **single resolver call** using the `--x86-only:` prefix:

```bash
# Single resolver call - x86-only files skipped on ARM64
/tmp/install-conda-deps.sh \
  /tmp/requirements/baseimage.txt \
  /tmp/requirements/core.txt \
  /tmp/requirements/classify.txt \
  --x86-only:/tmp/requirements/classify-x86.txt
```

This prevents version regressions. **Never install incrementally.**

The `install-conda-deps.sh` script:
- On x86: Includes all files in one micromamba call
- On ARM64: Skips files tagged with `--x86-only:` but includes others

---

## Docker Images

### Image Hierarchy

```
baseimage (conda/python)
    └── core (core tools)
        ├── assemble (+ assembly tools)
        ├── classify (+ classification tools)
        ├── phylo (+ phylo tools)
        └── mega (all tools)
```

### Tag Format

```
quay.io/broadinstitute/viral-ngs:2.6.0-core
quay.io/broadinstitute/viral-ngs:2.6.0-classify
quay.io/broadinstitute/viral-ngs:2.6.0              # mega (no suffix)
quay.io/broadinstitute/viral-ngs:main-core          # main branch
quay.io/broadinstitute/viral-ngs:latest             # alias for main mega
```

### Building Locally

```bash
# Build baseimage
docker build -t viral-ngs:baseimage -f docker/Dockerfile.baseimage .

# Build core (needs baseimage)
docker build --build-arg BASEIMAGE=viral-ngs:baseimage \
  -t viral-ngs:core -f docker/Dockerfile.core .

# Build derivatives (need core)
docker build --build-arg BASEIMAGE=viral-ngs:core \
  -t viral-ngs:classify -f docker/Dockerfile.classify .
```

---

## CI/CD

### GitHub Actions Workflow

The `.github/workflows/docker.yml` workflow handles building and testing:

**Build Architecture:**
Each image flavor is built using 3 parallel jobs for native multi-arch support:
1. `build-{flavor}-amd64` - runs on `ubuntu-latest`
2. `build-{flavor}-arm64` - runs on `ubuntu-24.04-arm` (native ARM runner)
3. `create-manifest-{flavor}` - combines arch-specific images into multi-arch manifest

This approach is 3-5x faster than QEMU emulation for ARM builds.

**Build Job Flow:**
```
paths-filter + get-version (parallel)
         ↓
build-baseimage-amd64  ←→  build-baseimage-arm64  (parallel)
         ↓                          ↓
    create-manifest-baseimage
         ↓
build-core-amd64  ←→  build-core-arm64  (parallel)
         ↓                    ↓
    create-manifest-core
         ↓
build-{assemble,classify,phylo,mega}-amd64  ←→  build-{...}-arm64  (parallel)
         ↓
    create-manifest-{flavor}
         ↓
    test-{flavor} + test-{flavor}-arm64 (ARM64 tests only on PRs with docker changes)
         ↓
    deploy-to-quay (push/tag events only)
```

**Test Jobs:**
- **test-core**: Runs on x86, tests `tests/unit/core/`
- **test-assemble**: Runs on x86, tests `tests/unit/assemble/`
- **test-classify**: Runs on x86, tests `tests/unit/classify/`
- **test-phylo**: Runs on x86, tests `tests/unit/phylo/`
- **test-{flavor}-arm64**: Runs on native ARM, only on PRs when docker files change

**Smart Test Scoping:**
Tests only run when relevant code changes:
- Core tests: `src/viral_ngs/*.py`, `core/**`, `util/**`, `tests/unit/core/**`
- Assemble tests: `assemble/**`, `assembly.py`, or core changes
- Classify tests: `classify/**`, `metagenomics.py`, `taxon_filter.py`, or core changes
- Phylo tests: `phylo/**`, `interhost.py`, `intrahost.py`, `ncbi.py`, or core changes
- Docker changes trigger all tests (including ARM64 tests on PRs)

**Coverage:**
Each x86 test job uploads coverage to Codecov with flavor-specific flags.

### Multi-Architecture Support

- Images built natively for `linux/amd64` and `linux/arm64` using parallel runners
- Multi-arch manifests created with OCI annotations using `docker buildx imagetools create`
- x86-only packages (novoalign, mvicuna, bmtagger, kallisto, table2asn) handled via `--x86-only:` prefix in `install-conda-deps.sh`
- Python tool wrappers still importable on ARM64; only runtime execution fails for missing binaries
- Tests using x86-only tools have `@unittest.skipIf(IS_ARM, ...)` decorators
- Architecture-specific caches prevent cross-arch cache pollution

### ARM Test Skipping

Tests that use x86-only bioconda packages must be decorated to skip on ARM:

```python
from tests import IS_ARM

SKIP_X86_ONLY_REASON = "tool requires x86-only bioconda package (not available on ARM)"

@unittest.skipIf(IS_ARM, SKIP_X86_ONLY_REASON)
class TestSomeTool(TestCaseWithTmp):
    ...

# Or at method level:
@unittest.skipIf(IS_ARM, SKIP_X86_ONLY_REASON)
def test_specific_tool(self):
    ...
```

### Documentation Build

The `docs.yml` workflow builds Sphinx documentation. Key points:
- Uses `mock` to stub heavy dependencies (`Bio`, `pysam`, `scipy`, etc.) in `docs/conf.py`
- When adding new imports to source code, add corresponding mocks to `MOCK_MODULES` in `docs/conf.py`
- Runs `sphinx-build -W` (warnings as errors)

### Registry Strategy

- **GHCR (ghcr.io)**: Primary build registry, images pushed during CI for all events including PRs
- **Quay.io**: Production registry, images copied from GHCR after tests pass (push/tag events only)
- Feature branch images should be cleaned up periodically from Quay.io

---

## Coding Guidelines

### Agent Attribution

**Commit messages**: By default, do NOT include agent/model credits (e.g., "Co-Authored-By: Claude") in commit messages. This reduces noise in the git history.

**Code review comments**: DO include notes about agent/model involvement when writing code review comments (e.g., PR reviews, inline comments). This provides useful context about how the review was conducted.

**Explicit requests**: Include agent attribution in commits or elsewhere when explicitly requested by a human reviewer or contributor.

**Avoid amending pushed commits**: Do not use `git commit --amend` after a commit has been pushed to the remote. Amending pushed commits causes problems for collaboration. Instead, create a new commit with the fix. Amending is fine for local commits that haven't been pushed yet.

### Test-Driven Development

1. Write tests first
2. Verify tests fail
3. Implement feature
4. Verify tests pass
5. Refactor if needed

### Avoid Over-Engineering

- Only make changes directly requested
- Don't add features beyond what was asked
- Don't add comments/docstrings to unchanged code
- Don't create abstractions for one-time operations

### Security

- Never introduce command injection, XSS, SQL injection vulnerabilities
- Validate at system boundaries (user input, external APIs)
- Trust internal code and framework guarantees

### Code Style

- Use explicit imports (prefer `import x.y.z` over `from x.y import z`)
- Follow existing patterns in the codebase
- Run tests before committing

---

## Common Operations

### Verify Python imports

```bash
docker run --rm viral-ngs:core python -c "
import viral_ngs.core
import viral_ngs.core.samtools
import viral_ngs.core.picard
import viral_ngs.core.file
import viral_ngs.core.misc
print('Core imports OK')
"
```

### Run syntax check on all files

```bash
find src tests -name "*.py" -exec python -m py_compile {} \;
```

### Check ARM64 package availability

```bash
micromamba search -c bioconda <package> --subdir linux-aarch64
```

---

## Reusable Agent Skills

Established workflows and playbooks live in `.agents/skills/`. Each skill
directory contains a `SKILL.md` with the playbook and companion scripts.
Check there before building analysis pipelines from scratch.

Available skills:
- **regression-testing** -- End-to-end assembly regression testing against Terra submissions
- **dsub-batch-jobs** -- Running one-off compute jobs on GCP via dsub
- **container-vulns** -- Container vulnerability scanning, triage, and mitigation
- **claude-on-vertex-ci** -- Invoking Claude (via claude-code-action) on Vertex AI from
  GitHub Actions workflows. Covers the WIF infra in `viral-seq-ai`, how to add a new
  Claude-in-CI use case, the canonical workflow YAML pattern, and gotchas.

---

## Container Vulnerability Management

See `.agents/skills/container-vulns/SKILL.md` for detailed guidance on vulnerability
scanning, triaging CVEs, Rego policy, and ARM64 solver conflicts.

---

## Troubleshooting

### Circular Import Errors

- Use relative imports within packages (`from . import X`)
- Check import order in `__init__.py` files
- Don't import parent package from child modules

### Missing Tool Errors

- Verify tool is in the appropriate requirements file
- Check if tool is x86-only (add to `core-x86.txt`)
- Tool wrappers should fail gracefully at runtime, not import time

### Docker Build Failures

- Check that BASEIMAGE arg points to existing image
- Verify all requirements files exist
- Check for conda resolution conflicts (install all deps together)

---
> Source: [broadinstitute/viral-ngs](https://github.com/broadinstitute/viral-ngs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
