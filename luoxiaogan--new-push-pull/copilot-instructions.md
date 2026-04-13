## new-push-pull

> Generates communication topology matrices for distributed optimization.

# Distributed Optimization Simulation Project

## 🔴 CRITICAL RULES - MUST FOLLOW

### Plan Tool Execution Rule
**IMPORTANT**: When using the Plan tool or entering plan mode:
1. **ALWAYS** save the generated plan as a new `.md` file in `.claude/tasks/` directory
2. **File naming**: Use format `YYYY-MM-DD_HH-MM_task_description.md` 
3. **File content**: Include the complete plan with clear steps, objectives, and success criteria
4. **Execution**: After saving the plan file, follow the saved plan systematically
5. **Reference**: Always reference the saved plan file path when executing tasks

Example workflow:
```
User requests → Enter plan mode → Create plan → Save to .claude/tasks/2024-XX-XX_XX-XX_task_name.md → Execute according to saved plan
```

This ensures all plans are documented, traceable, and can be reviewed or resumed later.

## Project Overview

This project simulates distributed optimization algorithms on a single GPU. It is designed to test and analyze various distributed optimization strategies, particularly the PushPull algorithm, without requiring an actual distributed computing environment.

**Key Features:**
- Single GPU simulation of multi-node distributed training
- Support for heterogeneous data distribution
- Graph-based communication topology
- Different learning rate strategies
- Support for MNIST and CIFAR10 datasets

## Project Structure

```
new_push_pull/
├── data/                    # Training data directory (local: empty, server: contains MNIST/CIFAR10)
├── datasets/                # Data processing and splitting scripts for distributed optimization
│   ├── __init__.py
│   └── prepare_data.py      # Handles data partitioning for heterogeneous distribution
├── models/                  # Model architectures
│   ├── __init__.py
│   ├── cnn.py              # CNN model implementation
│   └── fully_connected.py  # Fully connected network implementation
├── training/               # Training loops and optimizers
│   ├── optimizer_push_pull_grad_norm_track.py
│   ├── optimizer_push_pull_grad_norm_track_different_learning_rate.py
│   ├── training_track_grad_norm.py
│   └── training_track_grad_norm_different_learning_rate.py
├── utils/                  # Utility functions
│   ├── algebra_utils.py    # Matrix operations (Perron vectors, etc.)
│   ├── network_utils.py    # Graph/network topology utilities
│   └── train_utils.py      # Training helper functions
├── scripts_pushpull_differ_lr/  # Experiment execution scripts
│   ├── Nearest_neighbor_MNIST.py  # Example training script
│   └── network_utils.py          # Network generation utilities (project-specific)
└── NEW_PROJECT_20250717/   # Output directory for CSV results and analysis
```

## Key APIs and Functions

### Training Functions

**`train_track_grad_norm_with_hetero_different_learning_rate()`**
Main training function with heterogeneous data distribution and different learning rates per node.

```python
from training import train_track_grad_norm_with_hetero_different_learning_rate

df = train_track_grad_norm_with_hetero_different_learning_rate(
    algorithm="PushPull",         # Algorithm type
    lr_list=[...],               # List of learning rates for each node
    A=A,                         # Row-stochastic mixing matrix
    B=B,                         # Column-stochastic mixing matrix
    dataset_name="MNIST",        # Dataset: "MNIST" or "CIFAR10"
    batch_size=128,             # Batch size
    num_epochs=500,             # Number of epochs
    remark="experiment_name",    # Experiment identifier
    alpha=1000,                 # Heterogeneity parameter (higher = more uniform)
    root="/path/to/output",     # Output directory
    use_hetero=True,            # Enable heterogeneous data distribution
    device="cuda:0",            # GPU device
    seed=42                     # Random seed
)
```

### Network/Graph Utilities

**`generate_nearest_neighbor_matrices()`**
Generates communication topology matrices for distributed optimization.

```python
from network_utils import generate_nearest_neighbor_matrices

A, B = generate_nearest_neighbor_matrices(
    n=16,    # Number of nodes
    k=3,     # Number of neighbors per node
    seed=42  # Random seed
)
```

### Perron Vector Computation

```python
from utils import get_left_perron, get_right_perron

pi_a = get_left_perron(A)   # Left Perron vector of matrix A
pi_b = get_right_perron(B)  # Right Perron vector of matrix B
```

## Technical Details

### Communication Topology
- **Matrix A**: Row-stochastic mixing matrix (used for averaging)
- **Matrix B**: Column-stochastic mixing matrix (used for gradient tracking)
- Both matrices define the communication graph structure

### Learning Rate Strategies
1. **Uniform**: All nodes use the same learning rate
2. **Pi_a inverse**: Learning rates proportional to inverse of left Perron vector of A
3. **Pi_b inverse**: Learning rates proportional to inverse of right Perron vector of B

### Heterogeneous Data Distribution
- Controlled by `alpha` parameter:
  - Higher alpha → more uniform distribution
  - Lower alpha → more heterogeneous distribution
- Implemented via Dirichlet distribution

## Example Usage

```python
import sys
import os
import numpy as np

# Add project root to path
current_dir = os.path.dirname(os.path.abspath(__file__))
project_root = os.path.abspath(os.path.join(current_dir, '..'))
if project_root not in sys.path:
    sys.path.insert(0, project_root)

from training import train_track_grad_norm_with_hetero_different_learning_rate
from utils import get_left_perron, get_right_perron
from network_utils import generate_nearest_neighbor_matrices

# Configuration
n = 16                # Number of nodes
lr_basic = 7e-3      # Base learning rate
num_epochs = 100     # Training epochs
batch_size = 128     # Batch size
alpha = 1000         # Heterogeneity parameter
device = "cuda:0"    # GPU device

# Generate communication topology
A, B = generate_nearest_neighbor_matrices(n=n, k=3, seed=42)

# Compute Perron vectors
pi_b = get_right_perron(B)

# Set up learning rates (uniform strategy)
lr_total = lr_basic * n
lr_list = [lr_total / n] * n  # Uniform distribution

# Train model
df = train_track_grad_norm_with_hetero_different_learning_rate(
    algorithm="PushPull",
    lr_list=lr_list,
    A=A,
    B=B,
    dataset_name="MNIST",
    batch_size=batch_size,
    num_epochs=num_epochs,
    remark="uniform_lr_experiment",
    alpha=alpha,
    root="./output",
    use_hetero=True,
    device=device,
    seed=0
)

# Save results
df.to_csv("experiment_results.csv")
```

## Documentation References

### Synthetic Data Testing (合成数据测试)
For questions related to `合成数据_不使用cupy_最简单的版本/basic_test.py` or just the dir `合成数据_不使用cupy_最简单的版本/`, please refer to the detailed analysis document:
- **[Basic Test Analysis](.claude/documentation/basic_test_分析总结.md)**: Comprehensive analysis of the basic_test.py script and all its dependencies, including PushPull algorithm implementation, heterogeneous data distribution, and convergence analysis.

## Important Notes

1. **Local vs Server**: This is a local simulation project. Data paths are configured for server environments but work locally with empty data directories.

2. **GPU Memory**: When simulating many nodes, GPU memory usage scales with the number of nodes. Adjust batch size and number of nodes accordingly.

3. **Output Files**: Training results are saved as CSV files containing loss, gradient norms, and other metrics for each epoch.

4. **Reproducibility**: Always set seeds for reproducible experiments.

5. **Network Topology**: The choice of communication graph (A, B matrices) significantly affects convergence. Ensure matrices satisfy required properties (row/column stochasticity).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luoxiaogan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Luoxiaogan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
