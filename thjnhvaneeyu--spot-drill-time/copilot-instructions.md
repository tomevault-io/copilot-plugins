## spot-drill-time

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a comprehensive collection of optimization algorithms for **Arc Proton Therapy** treatment planning, focusing on spot drill time optimization and dose delivery efficiency. The codebase implements multiple mathematical norm approaches based on research by Sophie Wuyckens et al.

**Author**: Huỳnh Khôi Minh Uyên  
**Last Updated**: 2025-08-15  
**Status**: Complete analysis with 21 algorithms implemented and validated

## Common Development Commands

### Running Individual Algorithms
```bash
# FISTA algorithms  
python fista_proton_optimization_demo.py         # Quick demonstration
python fista_proton_optimization_improved.py     # Advanced features

# ADMM variants
python ADMM_L1.py                    # L1 regularization (high sparsity)
python ADMM_L2.py                    # L2 regularization (smooth weights)
python ADMM_SoftThreshold.py         # Adaptive thresholding
python ADMM_L1_SoftThreshold.py      # Combined L1 + soft thresholding
python ADMM_L2_SoftThreshold.py      # L2 smoothness + selective sparsity

# Advanced norm implementations
python FISTA_L2_L1_GroupLasso.py     # Group Lasso (convex)
python ELOSPAT_L2_L1half_NonConvex.py # Non-convex L2,1/2 norm
python LocalSearch_L0_DirectSparsity.py # Direct L0 control
python SPArc_L0_Heuristic.py         # L0 heuristic approximation  
python MIP_L0_Exact.py               # Exact L0 via Mixed-Integer Programming
python ELSA_L0_L2_Hybrid.py          # Two-phase L0+L2 hybrid

# Main optimization wrappers
python SPArc_optimization.py
python LocalSearch_optimization.py
python MIP_optimization.py
python ELOSPAT_optimization.py
python ELSA_optimization.py
```

### Comprehensive Analysis and Visualization
```bash
# Complete analysis pipeline (RECOMMENDED)
python complete_visualization_pipeline.py   # Process all 21 algorithms with extended analysis

# Standard analysis pipeline  
python integrated_visualization_pipeline.py  # Process 17 core algorithms

# File standardization and analysis
python standardize_analyze_files.py         # Analyze algorithm implementations
```

### Algorithm Comparison
```bash
python algorithm_comparison.py       # Compare all implemented algorithms
```

### Testing and Validation
```bash
python test.py                      # Basic functionality tests
```

## Core Architecture

### Mathematical Framework
The repository implements multiple mathematical norm approaches for the general optimization problem:
```
minimize: f(d) + Regularization(x) + γ·ELST(x)
```
Where:
- `f(d) = ||Ax - d₀||²` (dose fidelity term)
- `Regularization(x)` varies by algorithm (L1, L2, L0, Group Lasso, etc.)
- `ELST(x)` is Energy Layer Switching Time penalty
- `A` is the kernel fluence matrix (voxels × spots)
- `x` are the spot weights to optimize

### Algorithm Categories

1. **FISTA-based algorithms** (`fista_*.py`): Fast proximal gradient methods
2. **ADMM variants** (`ADMM_*.py`): Alternating Direction Method of Multipliers  
3. **Local Search methods** (`LocalSearch_*.py`): Heuristic optimization with neighborhood operators
4. **Mixed-Integer Programming** (`MIP_*.py`): Exact L0 optimization with binary variables
5. **SPArc methods** (`SPArc_*.py`): Beam splitting and layer selection heuristics
6. **ELSA variants** (`ELSA_*.py`): Hybrid L0+L2 two-phase optimization
7. **ELOSPAT methods** (`ELOSPAT_*.py`): Non-convex L2,1/2 norm optimization
8. **Norm-specific implementations**: Each mathematical norm has dedicated implementation
9. **Optimization wrappers**: High-level interfaces for complete workflows

### Complete Algorithm Portfolio (21 Algorithms)

#### Standard Implementations (17)
- **ADMM Family**: L1, L2, SoftThreshold, L1_SoftThreshold, L2_SoftThreshold
- **FISTA Family**: Demo, Improved, L2_L1_GroupLasso  
- **Local Search**: Standard, L0_DirectSparsity
- **MIP**: Standard, L0_Exact
- **SPArc**: Standard, L0_Heuristic
- **ELSA**: Standard, L0_L2_Hybrid
- **ELOSPAT**: L2_L1half_NonConvex

#### Algorithm Variants (4)
- **Alternative**: ELOSPAT_Alternative (alternative L2,1/2 implementation)
- **Original**: FISTA_Demo_Original, FISTA_Improved_Original (original versions)
- **Reduced**: ELSA_L2_Reduced (reduced-dimension model)

### Data Requirements
All algorithms require:
- `Kernel_targetPID9174251.mat`: MATLAB file containing sparse matrix `Target_fluence` (105 voxels × 4934 spots)
- Prescribed dose (typically 1.8 Gy)

### Algorithm Output Files
Each algorithm generates standardized `.npy` files:
- **Standard outputs**: `[algorithm]_optimal_weights.npy`
- **Special analysis**: `mip_l0_exact_binary.npy`, `elsa_l0_selection_mapping.npy`
- **Variants**: `[algorithm]_[variant]_weights.npy`

### Results and Analysis
The repository includes comprehensive analysis infrastructure:

#### Analysis Directories
- **`integrated_results/`**: Standard 17-algorithm analysis
- **`complete_results/`**: Extended 21-algorithm analysis with variants
- **Individual plots**: Per-algorithm detailed visualizations
- **Comparison dashboards**: Multi-algorithm performance analysis
- **CSV reports**: Quantitative comparison data

#### Key Analysis Files
- **`README.md`**: Comprehensive analysis documentation in `complete_results/`
- **`ADMM_OPTIMIZATION_STRATEGIES.md`**: Advanced ADMM optimization strategies
- **Dashboard PNGs**: Multi-panel algorithm comparison visualizations
- **Individual result PNGs**: 3×3 extended analysis per algorithm

### Key Architectural Components

#### Spot Drill Time (SDT) Model
All algorithms use the IBA P+ proton machine model:
```python
SDT(w) = w * (2588.986 / ((w + 1.256)^16.721) + 4.995)
```

#### Energy Layer Switching Time (ELST)
Penalty for energy transitions:
```python
ELST = Σ (switch_up_penalty × 5.5s + switch_down_penalty × 0.6s)
```

#### Arc Structure
- **46 beams** with continuous gantry rotation
- **10 energy layers per beam** 
- Variable spots per beam (typically ~107 spots/beam)

### Algorithm-Specific Features

#### L0 Norm Implementations
- **Direct Control** (`LocalSearch_L0_DirectSparsity.py`): 6 specialized neighborhood operators
- **Heuristic** (`SPArc_L0_Heuristic.py`): Greedy beam splitting and layer selection
- **Exact** (`MIP_L0_Exact.py`): Mixed-Integer Programming with binary variables
- **Hybrid** (`ELSA_L0_L2_Hybrid.py`): Two-phase L0 selection + L2 optimization

#### Non-Convex Approaches
- **L2,1/2 norm** (`ELOSPAT_L2_L1half_NonConvex.py`): Enhanced sparsity through fractional powers
- **Local Search**: Simulated annealing with adaptive operators

#### Convex Methods
- **Group Lasso** (`FISTA_L2_L1_GroupLasso.py`): Energy layer group selection
- **ADMM variants**: Different regularization combinations

### Common Solution Structure
All algorithms produce:
- **Optimal weights** (`.npy` files): Spot intensity array (4934 elements)
- **Visualization plots** (`.png` files): 3×3 extended analysis plots
- **Convergence history**: Objective function evolution
- **Clinical metrics**: Dose error, sparsity percentage, ELST, SDT, delivery time

### Clinical Performance Summary
Based on comprehensive analysis of all 21 algorithms:

#### Top Performers by Category
| Metric | Algorithm | Value | Clinical Assessment |
|--------|-----------|--------|-------------------|
| **Best Sparsity** | FISTA_L2_L1_GroupLasso | 99.1% (43 active spots) | Ultra-sparse but slow |
| **Best Dose Accuracy** | ELOSPAT_Alternative | 0.029 Gy error | Excellent but slow |
| **Fastest Delivery** | SPArc_L0_Heuristic | 2,350s | Good for emergencies |
| **Best Clinical Balance** | LocalSearch | 95.2% sparse, 0.074 Gy error | **Recommended for routine use** |
| **Most Stable** | ADMM_SoftThreshold | 9/10 stability | Research preferred |

#### Clinical Recommendations
1. **Routine treatments**: LocalSearch (best overall balance)
2. **Emergency/fast cases**: SPArc variants (~40 minutes)
3. **Dose-critical cases**: ELOSPAT_Alternative (premium quality)
4. **Research studies**: ADMM_SoftThreshold (excellent stability)
5. **Avoid clinically**: Dense algorithms (ADMM variants, FISTA demos)

### File Naming Conventions
- `*_optimization.py`: High-level algorithm wrappers
- `*_L0_*.py`: L0 norm specific implementations
- `*_L1_*.py`: L1 norm variants
- `*_L2_*.py`: L2 norm variants  
- `ADMM_*.py`: ADMM algorithm family
- `FISTA_*.py`: FISTA algorithm family
- `complete_visualization_pipeline.py`: **MAIN ANALYSIS SCRIPT** (21 algorithms)
- `integrated_visualization_pipeline.py`: Standard analysis (17 algorithms)
- `standardize_analyze_files.py`: Algorithm implementation analysis

### Analysis Pipeline Outputs
- **Timestamp format**: `YYYYMMDD_HHMMSS_[description].[ext]`
- **Individual results**: `[timestamp]_[algorithm]_extended_results.png`
- **Dashboard**: `[timestamp]_complete_dashboard.png`
- **CSV data**: `[timestamp]_complete_comparison.csv`
- **Documentation**: `README.md`, `ADMM_OPTIMIZATION_STRATEGIES.md`

### Performance Considerations
- **Memory**: Kernel matrix (105×4934) loaded into memory for all algorithms
- **Computation**: Non-convex and L0 methods are computationally intensive
- **Convergence**: Adjust `max_iterations` and tolerance based on quality requirements
- **Analysis time**: Complete pipeline takes ~10-15 minutes for all 21 algorithms

### Known Issues and Fixes Applied
1. **MIP_optimization.py**: Fixed LNS infeasibility issues by implementing greedy initialization from MIP_L0_Exact.py
2. **SPArc_L0_Heuristic.py**: Fixed AttributeError by reordering initialization sequence  
3. **Visualization**: Fixed Excel export issues by adding openpyxl error handling
4. **File size mismatches**: Added padding/truncation logic for different .npy file sizes
5. **FISTA_L2_L1_GroupLasso**: Extremely long execution time (116M+ seconds) - avoid for practical use

### Development Tips
- **Start with**: `complete_visualization_pipeline.py` for full analysis
- **Individual testing**: Run specific algorithm files for debugging
- **Results location**: Check `complete_results/` for latest comprehensive analysis
- **Performance comparison**: Use CSV files for quantitative analysis
- **Visualization**: PNG files provide clinical interpretation
- **ADMM optimization**: See `ADMM_OPTIMIZATION_STRATEGIES.md` for advanced parallel strategies

### Medical Context
This is research code for **Arc Proton Therapy** treatment planning optimized for **IBA Proteus Plus** systems. Key clinical objectives:
1. **Dose fidelity**: Achieve prescribed dose distribution (target: 1.8 Gy)
2. **Sparsity**: Minimize active spots for faster delivery (target: 80-95% sparsity)
3. **Energy sequencing**: Minimize energy layer switching time (ELST)
4. **Delivery efficiency**: Reduce total treatment time (<1 hour ideal)

### Clinical Context
- **Target anatomy**: Based on patient ID 9174251 
- **Treatment parameters**: 46 beam angles, ~107 spots per beam, 4934 total spots
- **Dose prescription**: 1.8 Gy target dose with ±5% tolerance
- **Delivery constraints**: IBA P+ spot drill time model + energy switching penalties

### Research vs Clinical Perspective
The analysis reveals important differences between research and clinical preferences:
- **Research preferred**: ADMM_SoftThreshold (excellent mathematical stability)
- **Clinically preferred**: LocalSearch (best balance of dose accuracy + sparsity + delivery time)
- **Emergency use**: SPArc (fastest delivery, acceptable dose quality)
- **Premium cases**: ELOSPAT_Alternative (best dose accuracy, high sparsity)

**Important**: These algorithms are for research purposes only and not validated for clinical use. Clinical implementation requires extensive validation and regulatory approval.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thjnhvaneeyu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
