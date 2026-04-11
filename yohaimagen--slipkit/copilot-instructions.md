## slipkit

> Google-style docstrings, type hinting, and pytest for testing


# SlipKit: Earthquake Slip Inversion Package - Design Specification

## General Instructions
Google-style docstrings, type hinting, and pytest for testing
ech Stack: Explicitly list numpy, scipy, cutde, pandas, pyproj, and meshio

## 1. Project Overview

**Goal:** Build a high-performance, modular Python package for static earthquake fault slip inversion ($Gm = d$).
**Philosophy:** Strict separation of concerns. The solver is agnostic to physics; the physics engine is agnostic to data formats; data containers are agnostic to inversion logic.
**Core Standards:**

* **Physics Backend:** `cutde` (Triangle Dislocation Elements) for elastic Green's functions (CPU/GPU).

* **Coordinate System:** Local Cartesian (meters).

* **Regularization:** Laplacian smoothing on unstructured meshes.

* **Extensibility:** Abstract base classes for Solvers, Fault Models, and Green's Functions to allow user-defined implementations.

## 2. System Architecture

The system is composed of five distinct layers:

1. **Data Layer:** Generic containers for observations (`GeodeticDataSet`).

2. **Model Layer:** Geometric representations of faults (`AbstractFaultModel`).

3. **Physics Layer:** Computation of elastic responses (`GreenFunctionEngine`).

4. **Assembly Layer:** Construction of the linear system $[G; \lambda L]$ and $[d; 0]$.

5. **Solver Layer:** Numerical optimization strategies (`SolverStrategy`).

## 3. Module & Class Definitions

### 3.1. `core.data`

#### `GeodeticDataSet`

A generic container for observed displacements. It is agnostic to the source (InSAR vs. GNSS).

* **Attributes:**

  * `coords`: `(N, 3) np.ndarray` - Observation points $(x, y, z)$ in local coordinates.

  * `data`: `(N,) np.ndarray` - Observed displacement values (scalar).

  * `unit_vecs`: `(N, 3) np.ndarray` - Unit vectors for projection (e.g., LOS or E/N/U).

  * `sigma`: `(N,) np.ndarray` - Data uncertainties (diagonal covariance).

  * `name`: `str` - Identifier (e.g., "Sentinel1_Asc").

* **Methods:**

  * `get_nuisance_basis()`: Returns `None` for V1. Future-proof hook for InSAR orbital ramps (returning matrix of shape `(N, k)`).

  * `__len__()`: Returns $N$.

### 3.2. `core.fault`

#### `AbstractFaultModel` (ABC)

The interface for any fault geometry.

* **Abstract Methods:**

  * `num_patches()`: Returns total number of sub-faults ($M$).

  * `get_mesh_geometry()`: Returns vertices and faces (or equivalent geometric description).

  * `get_centroids()`: Returns `(M, 3)` coordinates of patch centers (for visualization).

  * `get_smoothing_matrix(type='laplacian')`: Returns the sparse $(M, M)$ regularization matrix $L$ based on topology.

#### `TriangularFaultMesh(AbstractFaultModel)`

Implementation for unstructured triangular meshes.

* **Initialization:** Accepts `.msh` or `.stl` file path, or raw vertices/faces arrays.

* **Logic:**

  * Builds an adjacency graph of triangles.

  * Computes the Laplacian $L$ where $L_{ij} = -1$ if $j$ is neighbor of $i$, and $L_{ii} = \text{degree}(i)$.

### 3.3. `core.physics`

#### `GreenFunctionBuilder` (ABC)

Interface for calculating the elastic response matrix $G$.

* **Abstract Methods:**

  * `build_kernel(fault: AbstractFaultModel, data: GeodeticDataSet)`: Returns the generic Green's function matrix $G$.

#### `CutdeCpuEngine(GreenFunctionBuilder)`

The standard implementation using `cutde`.

* **Configuration:** Poisson's ratio, Young's modulus.

* **Logic:**

  * Iterates over `fault` patches and `data` points.

  * Calls `cutde.disp_matrix`.

  * **Crucial:** Computes two responses per patch (Strike-Slip unit, Dip-Slip unit).

  * **Output:** Returns matrix of shape $(N_{data}, 2 \times M_{patches})$.

    * Columns $0 \dots M-1$: Strike-Slip response.

    * Columns $M \dots 2M-1$: Dip-Slip response.

### 3.4. `core.regularization`

#### `RegularizationManager` (ABC)

Abstract base class that defines the strategy for constructing the regularization matrix $L$. This ensures the architecture can support various smoothing techniques (Laplacian, Gradient, Minimum Moment) and custom logic for handling multiple interacting faults.

* **Abstract Methods:**

  * `build_smoothing_matrix(faults: List[AbstractFaultModel], lambda_spatial: float)`: Returns the global sparse regularization matrix $S$.

#### `LaplacianSmoothing(RegularizationManager)`

The standard implementation using topological Laplacian operators.

* **Logic:**

  * Iterates over the list of `faults`.

  * Calls `fault.get_smoothing_matrix()` for each fault.

  * Constructs the global block-diagonal matrix for the full system (assuming independent faults by default).

  * Handles the 2-component logic:

    $$
    S_{full} = \begin{bmatrix} \lambda L & 0 \\ 0 & \lambda L \end{bmatrix}
    $$

    (Smooths strike-slip and dip-slip independently).

### 3.5. `core.solvers`

#### `SolverStrategy` (ABC)

Interface for numerical solvers.

* **Abstract Methods:**

  * `solve(A, b, bounds=None)`: Returns solution vector $m$ and fit metrics.

#### `NnlsSolver(SolverStrategy)`

Standard Non-Negative Least Squares.

* **Logic:** Wraps `scipy.optimize.nnls`.

* **Use Case:** Strictly for cases where slip direction is known (e.g., pure thrust).

#### `BoundedLsqSolver(SolverStrategy)`

Wraps `scipy.optimize.lsq_linear`.

* **Logic:** Allows specific bounds (e.g., $-\infty < \text{strike} < \infty$, $0 < \text{dip} < \infty$).

* **Use Case:** Variable rake or mixed-mode slip.

## 4. The Orchestrator (`core.inversion`)

#### `InversionOrchestrator`

The user-facing API that ties everything together.

* **Attributes:**

  * `faults`: List of `AbstractFaultModel`.

  * `datasets`: List of `GeodeticDataSet`.

  * `engine`: Instance of `GreenFunctionBuilder`.

  * `solver`: Instance of `SolverStrategy`.

* **Methods:**

  * `add_fault(fault)`

  * `add_data(dataset)`

  * `set_engine(engine)`

  * `set_solver(solver)`

  * `run_inversion(lambda_spatial)`:

    1. **Assembly:**

       * Collects all data $d$ into a single vector.

       * Uses `engine` to build $G_{elastic}$.

       * Uses `dataset.get_nuisance_basis()` (future) to append ramp columns.

       * Uses `RegularizationManager` to build $S$.

       * Stacks the system: $A = \begin{bmatrix} \Sigma^{-1}G \\ \lambda S \end{bmatrix}$, $b = \begin{bmatrix} \Sigma^{-1}d \\ 0 \end{bmatrix}$.

    2. **Solve:** Calls `solver.solve(A, b)`.

    3. **Map:** Returns a `SlipDistribution` object (maps the raw $m$ vector back to the fault meshes).

## 5. Utilities (`utils`)

### 5.1. Parsers (`utils.parsers`)

* **`InsarParser`**:

  * Input: Raster file (GeoTiff), Line-of-Sight vector grid.

  * Action: Reads raster, flattens to 1D, assigns unit vectors, converts Lat/Lon to Local XY.

  * Output: `GeodeticDataSet`.

* **`GnssParser`**:

  * Input: CSV (Station, Lat, Lon, E, N, U, SigE, SigN, SigU).

  * Action: Converts to Local XY, builds the generic vectors.

  * Output: `GeodeticDataSet`.

### 5.2. Visualization (`utils.viz`)

* **`SlipVisualizer`**:

  * Input: `SlipDistribution` object.

  * Action: Plots 3D triangular mesh with color-coded slip magnitude and arrows for rake.

* **`ResidualVisualizer`**:

  * Input: `GeodeticDataSet` (observed) vs `PredictedData`.

  * Action: 2D scatter plots/maps of residuals.

## 6. Implementation Strategy

### Phase 1: The Core

1. Implement `AbstractFaultModel` and `TriangularFaultMesh`.

2. Implement `GeodeticDataSet`.

3. Implement `CutdeCpuEngine` (basic wrapper).

4. Implement `NnlsSolver`.

5. Implement basic `InversionOrchestrator` (no regularization yet).

### Phase 2: Regularization & Parsers

1. Implement `RegularizationManager` (Laplacian smoothing).

2. Add smoothing matrix assembly to `InversionOrchestrator`.

3. Implement `InsarParser` and `GnssParser`.

### Phase 3: Refinement

1. Add `BoundedLsqSolver` for variable rake.

2. Add Visualization tools.

3. Write Unit Tests for all core modules.

## 7. Future Roadmap

1. **Orbits/Ramps:** Implement `GeodeticDataSet.get_nuisance_basis()` to return polynomial ramp columns.

2. **Okada/Rectangular:** Create `RectangularFaultModel` and `OkadaCpuEngine`.

3. **Stitching:** Update `RegularizationManager` to smooth across fault boundaries.

4. **GPU:** Create `CutdeGpuEngine` leveraging CUDA.

## 8. Example Workflow

```python
from slipkit.utils.parsers import InsarParser, GnssParser
from slipkit.core.fault import TriangularFaultMesh
from slipkit.core.physics import CutdeCpuEngine
from slipkit.core.solvers import NnlsSolver
from slipkit.core.inversion import InversionOrchestrator
from slipkit.utils.viz import Visualizer

# 1. Setup Data
insar_data = InsarParser.read("track_103.tif", los_vecs=...)
gnss_data = GnssParser.read("stations.csv")

# 2. Setup Fault
mesh = TriangularFaultMesh("fault_geometry.msh")

# 3. Setup Physics
engine = CutdeCpuEngine(poisson=0.25)

# 4. Setup Inversion
inv = InversionOrchestrator()
inv.add_fault(mesh)
inv.add_data(insar_data)
inv.add_data(gnss_data)
inv.set_engine(engine)
inv.set_solver(NnlsSolver())

# 5. Run
# Internally builds G, builds Laplacian, stacks them, solves.
result = inv.run_inversion(lambda_spatial=0.5)

# 6. Viz
Visualizer.plot_slip(result)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yohaimagen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yohaimagen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
