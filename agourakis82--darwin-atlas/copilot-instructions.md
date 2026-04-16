## darwin-atlas

> **Name**: Darwin Operator Symmetry Atlas (DOSA)

# CLAUDE.md — Darwin Operator Symmetry Atlas

## Project Identity

**Name**: Darwin Operator Symmetry Atlas (DOSA)
**Version**: 2.0.0-alpha
**Principal Investigator**: Demetrios Chiuratto Agourakis
**Target Publication**: Scientific Data (Nature Portfolio) — Data Descriptor

---

## Mission Statement

Build a reproducible, DOI-versioned database of operator-defined symmetries in complete bacterial replicons, implementing a hybrid Demetrios + Julia architecture with cross-validation between implementations.

**NO PYTHON**. This project uses exclusively Julia (Layers 0-1) and Demetrios (Layer 2).

---

## Architecture Overview

```
Layer 3: Artifacts     → CSV/JSONL/Parquet (Zenodo DOI)
Layer 2: Demetrios     → High-performance kernels with epistemic computing
Layer 1: Julia         → Orchestration, NCBI fetch, validation
Layer 0: Julia Pure    → Reference implementation (fallback, cross-validation)
```

### Why This Architecture?

1. **Demetrios (Layer 2)**: Showcases the language's units of measure, refinement types, and epistemic computing for scientific applications
2. **Julia (Layers 0-1)**: Provides reproducibility guarantee for Scientific Data reviewers who may not have Demetrios installed
3. **Cross-validation**: Ensures both implementations produce **identical** results, catching bugs in either

---

## Directory Structure (Canonical)

```
darwin-atlas/
├── CLAUDE.md                     # THIS FILE - READ FIRST
├── README.md                     # Project documentation
├── Makefile                      # Build orchestration
├── .zenodo.json                  # DOI metadata
│
├── demetrios/                    # Layer 2: Demetrios Kernels
│   ├── demetrios.toml            # Project config
│   ├── src/
│   │   ├── lib.d                 # Library root, exports
│   │   ├── operators.d           # S/R/K/RC definitions with units
│   │   ├── exact_symmetry.d      # Fixed points, orbit ratio
│   │   ├── approx_metric.d       # d_min/L with refinement types
│   │   ├── quaternion.d          # Dic_n lift verification
│   │   └── ffi.d                 # C ABI exports for Julia
│   └── tests/
│
├── julia/                        # Layers 0 + 1
│   ├── Project.toml
│   ├── Manifest.toml             # MUST BE COMMITTED (reproducibility)
│   ├── src/
│   │   ├── DarwinAtlas.jl        # Module root
│   │   ├── Types.jl              # Shared type definitions
│   │   ├── Operators.jl          # Pure Julia operators (Layer 0)
│   │   ├── ExactSymmetry.jl      # Pure Julia exact symmetry
│   │   ├── ApproxMetric.jl       # Pure Julia approx metric
│   │   ├── QuaternionLift.jl     # Pure Julia quaternion
│   │   ├── NCBIFetch.jl          # NCBI download + manifest
│   │   ├── Validation.jl         # Technical validation suite
│   │   ├── DemetriosFFI.jl       # ccall wrappers (Layer 1→2)
│   │   └── CrossValidation.jl    # Demetrios vs Julia comparison
│   ├── test/
│   │   └── runtests.jl
│   └── scripts/
│       ├── run_pipeline.jl
│       └── cross_validation.jl
│
├── data/                         # Layer 3: Outputs
│   ├── raw/                      # Downloaded sequences (gitignored)
│   ├── manifest/
│   │   ├── manifest.jsonl
│   │   └── checksums.sha256
│   └── tables/
│       ├── atlas_replicons.csv
│       ├── atlas_windows_exact.csv
│       ├── approx_symmetry_stats.csv
│       ├── dicyclic_lifts.csv
│       └── quaternion_results.csv
│
├── paper/                        # Scientific Data manuscript
│   ├── main.tex
│   └── figures/
│
└── .github/workflows/ci.yml      # Automated testing
```

---

## Technical Specifications

### Operator Definitions (Mathematical Foundation)

| Symbol | Name | Definition | Group |
|--------|------|------------|-------|
| I | Identity | σ(i) = s_i | D_4 |
| R | Reverse | σ(i) = s_{n-1-i} | D_4 |
| K | Complement | σ(i) = complement(s_i) | D_4 |
| RC | Rev-Comp | σ(i) = complement(s_{n-1-i}) | D_4 |

### Data Schema (Canonical)

#### atlas_replicons.csv
| Field | Type | Constraint |
|-------|------|------------|
| assembly_accession | String | GCF_... format |
| replicon_id | String | Internal stable ID |
| replicon_type | Enum | {chromosome, plasmid, other} |
| length_bp | Int64 | > 0 |
| gc_fraction | Float64 | 0.0 ≤ x ≤ 1.0 |
| taxonomy_id | Int64 | NCBI taxid |
| checksum_sha256 | String | 64 hex chars |

#### atlas_windows_exact.csv
| Field | Type | Constraint |
|-------|------|------------|
| replicon_id | String | FK → atlas_replicons |
| window_length | Int64 | bp |
| window_start | Int64 | 0-indexed, circular |
| orbit_ratio | Float64 | 0.25 ≤ x ≤ 1.0 |
| is_palindrome_R | Bool | |
| is_fixed_RC | Bool | |
| orbit_size | Int64 | ∈ {1, 2, 4} |

#### approx_symmetry_stats.csv
| Field | Type | Constraint |
|-------|------|------------|
| replicon_id | String | |
| window_length | Int64 | |
| d_min | Float64 | ≥ 0 |
| d_min_over_L | Float64 | 0 ≤ x ≤ 1 |
| transform_family | Enum | {dihedral, RC, identity} |

#### dicyclic_lifts.csv
| Field | Type | Constraint |
|-------|------|------------|
| dihedral_order | Int64 | 4, 8, 16 |
| verified_double_cover | Bool | |
| lift_group | String | Dic_n notation |
| relations_satisfied | Bool | |

---

## Implementation Phases

### Phase 1: Foundation
- [ ] Julia `Project.toml` with all dependencies
- [ ] `Types.jl` — all data structures with validation
- [ ] `Operators.jl` — pure Julia R/K/RC operators
- [ ] Unit tests for operators
- [ ] Demetrios project scaffold

### Phase 2: Core Algorithms
- [ ] `ExactSymmetry.jl` — orbit computation, fixed points
- [ ] `ApproxMetric.jl` — d_min, baseline shuffle
- [ ] `QuaternionLift.jl` — Dic_n verification
- [ ] Demetrios implementations with FFI exports
- [ ] `DemetriosFFI.jl` — ccall wrappers

### Phase 3: Pipeline
- [ ] `NCBIFetch.jl` — download, manifest, checksums
- [ ] `run_pipeline.jl` — end-to-end orchestration
- [ ] `CrossValidation.jl` — implementation comparison
- [ ] `Validation.jl` — technical validation suite

### Phase 4: Outputs
- [ ] Generate all CSV tables
- [ ] Technical validation report
- [ ] Zenodo deposit preparation
- [ ] Scientific Data manuscript draft

---

## Coding Standards

### Julia
```julia
# Use BlueStyle formatting
# All exported functions need docstrings
# Concrete types, avoid Any
# Property-based testing where applicable

"""
    orbit_ratio(seq::LongDNA{4}) -> Float64

Compute orbit ratio: |orbit| / |D₄|.

# Returns
- 0.25 if orbit size is 1
- 0.5 if orbit size is 2
- 1.0 if orbit size is 4
"""
function orbit_ratio(seq::LongDNA{4})
    orbit_size(seq) / 4.0
end
```

### Demetrios
```d
// Use units of measure for physical quantities
// Use refinement types for domain constraints
// Explicit effect declarations
// FFI exports with #[export] #[no_mangle]

type OrbitRatio = { r: f64 | 0.25 <= r && r <= 1.0 }

pub fn orbit_ratio(seq: &DNASeq) -> OrbitRatio with Alloc {
    let size = orbit_size(seq) as f64
    size / 4.0
}
```

### Commits
```
feat: add quaternion lift verification
fix: correct circular window extraction
docs: update schema documentation
test: add property-based tests for operators
refactor: extract common validation logic
```

---

## Commands Reference

```bash
# Full build
make all

# Julia only
make julia

# Demetrios only
make demetrios

# Run tests
make test

# Cross-validation
make cross-validate

# Full pipeline
make pipeline

# Reproducibility check
make reproduce
```

### Julia REPL
```julia
using Pkg; Pkg.activate("julia")
Pkg.instantiate()  # First time only
Pkg.test()
include("julia/scripts/run_pipeline.jl")
```

---

## Critical Constraints

### Scientific Data Compliance
1. **NO RESULTS IN DATA DESCRIPTOR** — Methods + Data Records + Technical Validation only
2. **Data citations required** — DOI for all datasets
3. **Reproducibility** — Must work with `git clone` + `make reproduce`

### Cross-Validation Requirements
- Demetrios and Julia must produce **identical** outputs
- Tolerance: 0 for discrete values, 1e-12 for floating point
- **Any divergence is a blocking bug**

### Reproducibility Requirements
- All random seeds explicit and logged
- `Manifest.toml` committed (never gitignored)
- SHA256 checksums for all downloaded data
- Pipeline metadata JSON with versions, timestamps

---

## Error Handling Protocol

| Error Type | Action |
|------------|--------|
| Compilation error | Fix immediately, do not proceed |
| Test failure | Debug root cause, fix before continuing |
| Cross-validation divergence | **STOP**. This is critical. Debug until resolved |
| NCBI fetch failure | Retry with exponential backoff |
| Memory issue | Profile, optimize, or batch |

---

## Quality Gates

Before marking any phase complete:
- [ ] All unit tests pass
- [ ] No compiler warnings
- [ ] Documentation complete
- [ ] Cross-validation passes (if applicable)
- [ ] Self-review checklist complete

### Self-Review Checklist
- [ ] No hardcoded paths
- [ ] No magic numbers
- [ ] Error messages informative
- [ ] Edge cases handled
- [ ] Performance acceptable

---

## Target Scale

| Metric | Target |
|--------|--------|
| Replicons | ~50,000 complete bacterial genomes |
| Window sizes | 100, 500, 1000, 5000, 10000 bp |
| Processing time | < 24h on single node |
| Memory peak | < 64 GB (192 GB available) |
| GPU | L4 24GB + RTX 4000 Ada 20GB available |

---

## Communication Protocol

### Progress Updates (after each major component)
1. What was implemented
2. Test results summary
3. Deviations from plan
4. Next steps

### Blocking Issues (when stuck)
1. What is blocking
2. What was attempted
3. Proposed solutions
4. Decision needed

---

## Key Files to Reference

| File | Purpose |
|------|---------|
| `julia/src/Types.jl` | All type definitions |
| `julia/src/Operators.jl` | Reference implementation |
| `julia/test/runtests.jl` | Test suite entry |
| `demetrios/src/ffi.d` | FFI interface spec |
| `Makefile` | Build commands |

---

## External References

1. **SkewDB** — Template for Data Descriptor structure
2. **Scientific Data guidelines** — Data Descriptor format requirements
3. **Demetrios Language** — https://github.com/Chiuratto-AI/demetrios
4. **BioJulia docs** — BioSequences.jl, FASTX.jl
5. **NCBI Datasets API** — Data acquisition

---

## Initialization Command

To bootstrap this project, run:
```bash
bash init_project.sh darwin-atlas
cd darwin-atlas
julia --project=julia -e 'using Pkg; Pkg.instantiate()'
julia --project=julia -e 'using Pkg; Pkg.test()'
```

---

*Last updated: 2025-12-14*
*CLAUDE.md version: 1.0.0*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agourakis82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
