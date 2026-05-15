## bayesdream

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

bayesDREAM is a Bayesian framework for modeling perturbation effects across multiple molecular modalities. The model consists of three sequential steps:

1. **Technical fit** (`fit_technical`): Models technical variation in non-targeting controls (NTC) to estimate gene-specific overdispersion parameters (`alpha_y`)
2. **Cis fit** (`fit_cis`): Models direct effects on the targeted gene expression (`model_x`)
3. **Trans fit** (`fit_trans`): Models downstream effects on other genes as a function of the cis gene expression (`model_y`)

The codebase uses PyTorch and Pyro for probabilistic programming and variational inference.

## Repository Structure

```
bayesDREAM_forClaude/
├── bayesDREAM/
│   ├── __init__.py          # Package exports
│   ├── model.py             # Main bayesDREAM class (~311 lines)
│   ├── core.py              # _BayesDREAMCore base class (~909 lines)
│   ├── modality.py          # Modality class for multi-modal data
│   ├── distributions.py     # Distribution-specific observation samplers
│   ├── splicing.py          # Splicing data processing (pure Python)
│   ├── fitting/             # Fitting methods (modular)
│   │   ├── __init__.py
│   │   ├── helpers.py       # Shared helper functions
│   │   ├── technical.py     # TechnicalFitter class
│   │   ├── cis.py           # CisFitter class
│   │   └── trans.py         # TransFitter class
│   ├── io/                  # Save/load functionality
│   │   ├── __init__.py
│   │   ├── save.py          # ModelSaver class
│   │   └── load.py          # ModelLoader class
│   └── modalities/          # Modality-specific methods
│       ├── __init__.py
│       ├── transcript.py    # TranscriptModalityMixin
│       ├── splicing_modality.py  # SplicingModalityMixin
│       ├── atac.py          # ATACModalityMixin
│       └── custom.py        # CustomModalityMixin
├── tests/                   # Test suite
├── toydata/                 # Test datasets (genes, splicing, metadata)
└── docs/                    # Documentation
```

**Note**: The codebase was recently refactored from a single 4,537-line `model.py` file into a modular structure. This improves maintainability while preserving backward compatibility. See `docs/archive/planning/REFACTORING_SUMMARY.md` for details.

## Core Architecture

### bayesDREAM Class

The main class in `bayesDREAM/model.py` implements multi-modal Bayesian modeling with the three-step pipeline:

**Initialization:**
- Takes cell metadata DataFrame (`meta`) with columns: `cell`, `guide`, `cell_line`, `target`, `sum_factor`, etc.
- Takes counts DataFrame (`counts`) with genes as rows, cell barcodes as columns
- Optionally takes gene metadata DataFrame (`gene_meta`) with gene annotations
  - Recommended columns: `gene`, `gene_name`, `gene_id`
  - If not provided, minimal metadata is created from counts.index
  - Flexible identifier support: uses 'gene', 'gene_name', 'gene_id', or index
- Creates guide-level metadata by grouping cells by guide and specified covariates
- Supports both CPU and CUDA devices

**Key Methods:**

- `set_technical_groups(covariates)`: Sets technical_group_code based on covariates (must be called before fit_technical)
- `fit_technical(sum_factor_col, modality_name, ...)`: Fits NTC-only model to estimate `alpha_y_prefit`
- `set_alpha_x(alpha_x, is_posterior, covariates)`: Sets cis gene overdispersion parameters
- `set_alpha_y(alpha_y, is_posterior, covariates)`: Sets trans gene overdispersion parameters
- `adjust_ntc_sum_factor(covariates, ...)`: Adjusts NTC sum factors for covariates
- `fit_cis(sum_factor_col, ...)`: Fits cis effects using `_model_x`
- `set_x_true(x_true, is_posterior)`: Sets true cis expression for trans modeling
- `permute_genes(genes2permute, ...)`: Permutes guide-gene associations for null testing
- `refit_sumfactor(covariates, ...)`: Re-estimates sum factors based on posterior cis expression
- `fit_trans(sum_factor_col, function_type, modality_name, ...)`: Fits trans effects using `_model_y`

**Function Types for Trans Modeling:**

The `fit_trans` method supports multiple functional forms for modeling how trans gene expression depends on cis gene expression:

- `single_hill`: Single Hill equation (positive or negative)
- `additive_hill`: Additive combination of positive and negative Hill functions
- `polynomial`: Polynomial function with configurable degree (default: 6)

### Probabilistic Models

Three Pyro models implement the statistical framework:

1. **`_model_technical`**: Models NTC cells to estimate baseline overdispersion
   - Negative binomial likelihood with log-normal priors
   - Estimates per-gene `alpha_y` parameters

2. **`_model_x`**: Models cis effects on the targeted gene
   - Accounts for guide-level and cell-line-level variation
   - Estimates true gene expression `x_true` for each guide
   - Uses sum factors for normalization

3. **`_model_y`**: Models trans effects as functions of cis expression
   - Supports Hill-based functions or polynomials
   - Includes sparsity priors (gamma distribution on effect sizes)
   - Models gene-specific dose-response curves

## Common Development Tasks

### Testing

Run infrastructure tests (requires pyroenv conda environment):

```bash
/opt/anaconda3/envs/pyroenv/bin/python test_multimodal_fitting.py
```

### Modifying the Model

The codebase uses a modular structure with delegation:

1. **Core fitting logic**: Pyro models and fitting methods are in `fitting/` directory
   - `fitting/technical.py`: TechnicalFitter class with `_model_technical` and `fit_technical`
   - `fitting/cis.py`: CisFitter class with `_model_x` and `fit_cis`
   - `fitting/trans.py`: TransFitter class with `_model_y` and `fit_trans`
   - `fitting/helpers.py`: Shared helper functions (Hill functions, etc.)

2. **Base class**: `core.py` contains `_BayesDREAMCore` which:
   - Initializes fitter objects (`_technical_fitter`, `_cis_fitter`, `_trans_fitter`)
   - Delegates method calls to appropriate fitters
   - Contains utility methods (parameter setters, permutation, etc.)

3. **Main class**: `model.py` contains `bayesDREAM` which:
   - Inherits from `_BayesDREAMCore` and modality mixins
   - Handles multi-modal initialization
   - Provides modality management methods

4. **Modality mixins**: `modalities/` directory contains methods for adding different data types:
   - `transcript.py`: `add_transcript_modality`
   - `splicing_modality.py`: `add_splicing_modality`
   - `atac.py`: `add_atac_modality`
   - `custom.py`: `add_custom_modality`

5. **I/O operations**: `io/` directory contains save/load functionality
   - `save.py`: ModelSaver class
   - `load.py`: ModelLoader class

**When adding new functionality**:
- Helper functions → `fitting/helpers.py`
- New Pyro models → appropriate fitter in `fitting/` (follow `_model_<name>` convention)
- Modality-specific code → appropriate mixin in `modalities/`
- Save/load methods → `io/save.py` or `io/load.py`
- All fitters access model attributes via `self.model.*` (not `self.*`)

### Adding New Function Types

To add a new dose-response function:

1. Define the function in the helper section (e.g., `def my_function(x, params)`)
2. Add a conditional branch in `_model_y` to handle the new function type
3. Update `fit_trans` to set appropriate priors and optimization settings

### Testing Changes

The `toydata/` directory contains small test datasets. Use these for quick validation before running on full data:

- `gene_counts.csv`, `gene_meta.csv`: Gene expression data
- `SJ_counts.csv`, `SJ_meta.csv`: Splice junction data
- `SpliZ_counts.csv`, `SpliZ_meta.csv`: Splicing quantification

## Multi-Modal Architecture

bayesDREAM supports multiple molecular modalities beyond gene expression, allowing modeling of transcripts, splicing, and custom measurements within a unified framework.

### Modality Class

The `Modality` class (`bayesDREAM/modality.py`) provides a standardized container for different data types:

**Supported Distributions:**
- `negbinom`: Negative binomial (gene counts, transcript counts)
- `multinomial`: Categorical/proportional data (isoform usage, donor/acceptor usage)
- `binomial`: Binary outcomes with denominators (exon skipping PSI)
- `normal`: Continuous measurements (SpliZ scores)
- `studentt`: Heavy-tailed continuous (robust SpliZ)

**Data Structures:**
- **2D data** (negbinom, normal, studentt, binomial): `(features, cells)` or `(cells, features)`
- **3D multinomial**: `(features, cells, categories)` - e.g., donor sites × cells × acceptor options
- **Binomial**: 2D counts + 2D denominator array

**Key Features:**
- Feature-level metadata (gene names, junction coordinates, etc.)
- Cell subsetting and feature subsetting
- Automatic validation of shapes and distribution requirements
- Conversion to PyTorch tensors

### bayesDREAM Class (Multi-Modal)

The `bayesDREAM` class (`bayesDREAM/model.py`) provides full multi-modal support:

**Initialization:**
```python
from bayesDREAM import bayesDREAM

model = bayesDREAM(
    meta=cell_metadata,
    counts=gene_counts,              # Primary modality (genes)
    gene_meta=gene_metadata,         # Optional: gene annotations
    cis_gene='GFI1B',
    primary_modality='gene',         # Which modality drives cis/trans effects
    output_dir='./output',
    label='multimodal_run'
)
```

**Adding Modalities:**

1. **Transcript counts** (as counts and/or isoform usage):
   ```python
   # Add both counts and usage in one call
   model.add_transcript_modality(
       transcript_counts=tx_counts,
       transcript_meta=tx_meta,      # Must have: transcript_id + (gene/gene_name/gene_id)
       modality_types=['counts', 'usage']  # 'counts', 'usage', or both
   )

   # Or add just one type
   model.add_transcript_modality(
       transcript_counts=tx_counts,
       transcript_meta=tx_meta,
       modality_types='counts'       # Just negbinom counts
   )
   ```

2. **Splicing data** (raw SJ counts, donor/acceptor usage, exon skipping):
   ```python
   model.add_splicing_modality(
       sj_counts=sj_counts,
       sj_meta=sj_meta,              # Must have: coord.intron, chrom, intron_start, intron_end, strand, gene_name_start, gene_name_end
       splicing_types=['sj', 'donor', 'acceptor', 'exon_skip'],
       gene_counts=None,             # Optional: defaults to self.counts
       min_cell_total=1,             # Min reads for donor/acceptor
       min_total_exon=2              # Min reads for exon skipping
   )
   ```

3. **Custom modalities** (SpliZ, etc.):
   ```python
   # SpliZ scores (normal distribution)
   model.add_custom_modality(
       name='spliz',
       counts=spliz_scores,          # 2D: genes × cells
       feature_meta=gene_meta,
       distribution='normal'
   )
   ```

**Working with Modalities:**
```python
# List all modalities
print(model.list_modalities())

# Access specific modality
donor_mod = model.get_modality('splicing_donor')
print(donor_mod.dims)                    # {'n_features': 100, 'n_cells': 500, 'n_categories': 10}
print(donor_mod.feature_meta.head())     # Metadata: chrom, strand, donor, acceptors, etc.

# Subset modality
subset = donor_mod.get_feature_subset(['feature1', 'feature2'])
```

### Splicing Processing

The `splicing.py` module provides pure Python implementations for splicing analysis (no R dependencies):

**Raw SJ Counts** (`splicing_type='sj'`): Raw splice junction counts with gene expression as denominator.
- Distribution: `binomial`
- Numerator: SJ read counts (per-junction)
- Denominator: Gene-level expression (matched to each junction's gene)
- Dimensions: `(n_junctions, n_cells)`
- Metadata: All fields from SJ metadata, plus assigned `gene` identifier
- Note: Automatically filters to SJs with valid gene annotations

**Donor Usage** (`splicing_type='donor'`): Groups splice junctions by donor site (5' splice site). Returns multinomial counts showing which acceptor is used for each donor.
- Distribution: `multinomial`
- Dimensions: `(n_donors, n_cells, max_acceptors_per_donor)`
- Metadata: `chrom`, `strand`, `donor`, `acceptors` (list), `n_acceptors`

**Acceptor Usage** (`splicing_type='acceptor'`): Groups junctions by acceptor site (3' splice site). Returns multinomial counts showing which donor is used for each acceptor.
- Distribution: `multinomial`
- Dimensions: `(n_acceptors, n_cells, max_donors_per_acceptor)`
- Metadata: `chrom`, `strand`, `acceptor`, `donors` (list), `n_donors`

**Exon Skipping** (`splicing_type='exon_skip'`): Detects cassette exon triplets (inc1, inc2, skip) and computes inclusion/total counts.
- Distribution: `binomial`
- Dimensions: `(n_events, n_cells)` for both inclusion and total
- Metadata: `trip_id`, `chrom`, `strand`, `d1`, `a2`, `d2`, `a3`, `sj_inc1`, `sj_inc2`, `sj_skip`
- Methods: Strand-aware (default) or genomic coordinate fallback
- Aggregation: `min` (default) or `mean` for computing inclusion from inc1 and inc2

**SJ Metadata Requirements:**
- **Required columns:**
  - `coord.intron`: Junction ID (e.g., "chr1:12345:67890:+")
  - `chrom`: Chromosome
  - `intron_start`, `intron_end`: Junction coordinates
  - `strand`: Strand ('+', '-', 1, or 2)
  - `gene_name_start`: Gene name at junction start
  - `gene_name_end`: Gene name at junction end
- **Optional columns (for Ensembl ID support):**
  - `gene_id_start`: Ensembl gene ID at junction start
  - `gene_id_end`: Ensembl gene ID at junction end

**Gene Identifier Flexibility:**
- Works with gene names, Ensembl IDs, or both
- Priority for SJ-gene matching: `gene_name_start` → `gene_name_end` → `gene_id_start` → `gene_id_end`
- Tries all available identifiers when matching SJs to gene counts

### Current Limitations

1. **Cis/Trans fitting**: Currently only the primary modality (usually genes) is used for cis and trans modeling. Future versions will support modality-specific fits.

2. **Technical fitting**: Only the primary modality supports `fit_technical()`. Other modalities require manual specification of overdispersion parameters.

3. **Permutation testing**: `permute_genes()` operates on the primary modality.

4. **Sum factors**: Calculated only for gene-level data. Other modalities may need alternative normalization strategies.

### Example Workflows

**Comprehensive example**:
```python
from bayesDREAM import bayesDREAM

# Load data
meta = pd.read_csv('meta.csv')
gene_counts = pd.read_csv('gene_counts.csv', index_col=0)
sj_counts = pd.read_csv('SJ_counts.csv', index_col=0)
sj_meta = pd.read_csv('SJ_meta.csv')

# Create multi-modal model
model = bayesDREAM(
    meta=meta,
    counts=gene_counts,
    gene_meta=gene_meta,  # Optional: provide gene annotations
    cis_gene='GFI1B',
    output_dir='./output',
    label='multimodal_run'
)

# Add splicing modalities
model.add_splicing_modality(
    sj_counts=sj_counts,
    sj_meta=sj_meta,
    splicing_types=['sj', 'donor', 'acceptor', 'exon_skip']
)

# Inspect modalities
print(model.list_modalities())

# Set technical groups first (required before fit_technical)
model.set_technical_groups(['cell_line'])

# Run standard pipeline (operates on primary 'gene' modality)
model.fit_technical()
model.fit_cis(sum_factor_col='sum_factor')
model.fit_trans(sum_factor_col='sum_factor_adj', function_type='additive_hill')

# Access splicing data for downstream analysis
donor_modality = model.get_modality('splicing_donor')
donor_counts = donor_modality.counts        # 3D array
donor_meta = donor_modality.feature_meta    # Donor site annotations
```

See `tests/` directory for complete examples including transcripts, custom modalities, and advanced usage.

---

## Development Workflow

### Running Tests

All tests are in the `tests/` directory. Run them with:

```bash
cd "/Users/lrosen/Library/Mobile Documents/com~apple~CloudDocs/Documents/Postdoc/bayesDREAM code/bayesDREAM_forClaude"
export PYTHONPATH="."
/opt/anaconda3/envs/pyroenv/bin/python tests/test_multimodal_fitting.py
/opt/anaconda3/envs/pyroenv/bin/python tests/test_negbinom_compat.py
/opt/anaconda3/envs/pyroenv/bin/python tests/test_technical_compat.py
/opt/anaconda3/envs/pyroenv/bin/python tests/test_per_modality_fitting.py
/opt/anaconda3/envs/pyroenv/bin/python tests/test_gene_meta.py
/opt/anaconda3/envs/pyroenv/bin/python tests/test_modality_save_load.py
```

### Documentation

All documentation is in the `docs/` directory:

- `API_REFERENCE.md`: Public API documentation
- `ARCHITECTURE.md`: System architecture and design decisions
- `INITIALIZATION.md`: Technical fitting initialization strategies (empirical Bayes for negbinom, binomial, multinomial)
- `OUTSTANDING_TASKS.md`: **Current outstanding tasks and known issues**
- `SAVE_LOAD_GUIDE.md`: Guide to save/load functionality
- `QUICKSTART_MULTIMODAL.md`: Quick start for multi-modal analysis
- `SLURM_JOB_GENERATOR.md`: HPC job generation guide
- `MEMORY_REQUIREMENTS.md`: Memory estimation guide
- `PLOTTING_GUIDE.md`: Comprehensive visualization guide
- `SUMMARY_EXPORT_GUIDE.md`: Exporting results for R/plotting
- Various specialized guides (ATAC, splicing, high MOI, etc.)
- Historical planning documents available in `docs/archive/`

**Important**: Check `docs/OUTSTANDING_TASKS.md` for current development priorities, including the guide-prior infrastructure that needs to be integrated into `fit_cis()`.

### Code Organization

- **Python source**: `bayesDREAM/` directory
- **Tests**: `tests/` directory
- **Documentation**: `docs/` directory
- **Examples**: `examples/` directory
- **Test data**: `toydata/` directory

### Making Changes

1. Read `docs/OUTSTANDING_TASKS.md` to understand current priorities
2. Make changes in appropriate module (see "Modifying the Model" section above)
3. Run relevant tests to ensure nothing breaks
4. Update documentation if public API changes
5. Add tests for new functionality

---
> Source: [leahrosen/bayesDREAM](https://github.com/leahrosen/bayesDREAM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
