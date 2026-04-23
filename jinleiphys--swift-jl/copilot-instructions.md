## swift-jl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Julia-based nuclear physics framework implementing the Faddeev method for three-body quantum mechanical bound state calculations. The codebase specializes in solving nuclear three-body problems (like 3H tritium) using sophisticated numerical techniques including Laguerre basis functions, multi-channel coupling, and nuclear potential models.

## Development Commands

### Quick Setup (Automated)
For first-time setup or to update the environment, run the automated setup script:
```bash
./setup.sh
```
This script will:
1. Check Julia installation and install/update if needed (requires Julia >= 1.9.0)
2. Install all required Julia packages via `setup.jl`
3. Compile Fortran nuclear potential libraries in `NNpot/`

### Manual Setup (Alternative)
If you prefer manual setup or need to rebuild specific components:

**Julia Environment Setup:**
```bash
julia setup.jl
```

**Building Fortran Libraries:**
```bash
cd NNpot
make clean && make
```
This creates `libpotentials.dylib` (macOS), `libpotentials.so` (Linux), or `libpotentials.dll` (Windows).

### Build System Details
- **Fortran compiler**: `gfortran` with `-O2 -fPIC -Wall -Wextra` optimization
- **Platform detection**: Automatic selection of shared library format
- **F77/F90 compatibility**: Separate compilation flags for legacy and modern Fortran code
- **COULCC library**: Compiled automatically by `setup.sh` via `Makefile_coulcc` in `swift/`
  - Creates `libcoulcc.dylib` (macOS), `libcoulcc.so` (Linux), or `libcoulcc.dll` (Windows)
  - Provides Coulomb wavefunctions for scattering calculations

### Running Calculations
- **Main calculation**: `cd swift && julia swift_3H.jl` - Run 3H (tritium) bound state calculation
- **Script execution**: Run Julia files directly with `julia filename.jl`
- **Interactive development**: Use Jupyter notebooks in any subdirectory (*.ipynb files) for exploration
- **Memory-optimized runs**: Use `swift_3H_optimized.ipynb` for reduced memory calculations (~1-2 GB instead of 27 GB)

### Testing
- **Quick module test**: `julia NNpot/test.jl` - basic nuclear potential interface validation
- **Comprehensive test**: `julia NNpot/test_comprehensive.jl` - full system validation with multiple potentials
- **Physics validation**: `julia NNpot/test_channel_physics.jl` - channel coupling and quantum number consistency
- **Specific debugging**: Various `debug_*.jl` and `simple_*test*.jl` files for targeted testing

### Development Workflow
1. **Initial setup**: Run `./setup.sh` for automated environment setup (Julia installation, packages, Fortran libraries)
2. **Development**: Modify code, run calculations with `cd swift && julia swift_3H.jl`
3. **Interactive exploration**: Use Jupyter notebooks for debugging and method comparison
4. **Testing**: Run specific tests to validate changes (e.g., `julia NNpot/test_comprehensive.jl`)

## Core Architecture

### Module Structure
The codebase is organized into three main module directories:

1. **NNpot/**: Nuclear potential interface
   - Fortran libraries (AV18, AV14, Nijmegen) with Julia wrappers
   - `nuclear_potentials.jl`: Interface to compiled Fortran potentials
   - Dynamic library compilation via makefiles

2. **general_modules/**: Foundation components
   - `channels.jl`: Three-body channel coupling calculations with angular momentum algebra
   - `mesh.jl`: Laguerre-based numerical mesh generation for hyperspherical coordinates

3. **swift/**: Core Faddeev implementation
   - `matrices.jl`: Matrix elements for kinetic energy (T), potential (V), and coordinate transformations (Rxy)
   - `matrices_optimized.jl`: Optimized matrix computations with caching and complex scaling support
   - `scattering.jl`: Inhomogeneous scattering equation solver for three-body scattering calculations
   - `threebodybound.jl`: Direct eigenvalue solver for bound state energies
   - `MalflietTjon.jl`: Iterative Malfiet-Tjon eigenvalue solver with secant method convergence
   - `twobody.jl`: Two-body reference calculations (deuteron)
   - `laguerre.jl`: Basis function implementations
   - `Gcoefficient.jl`: Angular momentum coupling coefficients
   - `coulcc.jl`: Julia wrapper for COULCC Fortran library (Coulomb wavefunctions)

4. **3Npot/**: Three-body nuclear force models
   - `UIX.jl`: Urbana IX three-body force implementation with Y(r) and T(r) functions

### Key Physics Concepts
- **Faddeev equations**: Three-body quantum mechanics using coordinate transformations
- **Hyperspherical coordinates**: (x,y) representing relative distances in three-body system
- **Channel coupling**: Multi-channel approach with different angular momentum states
- **Nuclear potentials**: Realistic NN interactions (AV18, AV14, Nijmegen, Malfliet-Tjon)
- **Three-body forces**: Urbana IX (UIX) model with Y(r) and T(r) radial functions

### Computational Workflow

**Bound State Calculations:**
1. **Channel generation**: `α3b()` creates allowed quantum states based on conservation laws
2. **Mesh initialization**: `initialmesh()` sets up hyperspherical coordinate grids
3. **Matrix construction**: Build Hamiltonian H = T + V + V*Rxy using Faddeev rearrangement
   - Optional: Include three-body forces H₃ = T + V + V*Rxy + X₁₂ (UIX model)
4. **Eigenvalue solution**: Two approaches available:
   - **Direct method**: `ThreeBody_Bound()` solves generalized eigenvalue problem `eigen(H, B)`
   - **Iterative method**: `malfiet_tjon_solve()` uses secant iteration to find λ(E) = 1

**Scattering Calculations:**
1. **Initial state setup**: `compute_initial_state_vector()` creates source state (e.g., deuteron + nucleon)
   - Computes initial wavefunction φ = [φ_d(x) F_λ(ky)] / [ϕx ϕy]
   - Uses COULCC library for Coulomb functions F_λ
   - Supports complex scaling (θ≠0) for resonance calculations
2. **Matrix assembly**: `compute_scattering_matrix()` builds A = E*B - T - V*(I + Rxy)
   - Returns component matrices for M^{-1} preconditioner construction
3. **Source term**: `compute_VRxy_phi()` computes b = 2*V*Rxy_31*φ
   - Factor of 2 from Faddeev symmetry (two equivalent rearrangement channels)
   - Optimized multiplication order: V * (Rxy_31 * φ) for efficiency
4. **Linear solve**: `solve_scattering_equation()` solves [A]c = b for scattering wavefunction
   - **LU method**: Direct factorization for small systems
   - **GMRES method**: Preconditioned iterative solver with M^{-1} = [E*B - T - V_αα]^{-1}
     - Uses diagonal potential V_αα only (within-channel coupling)
     - Same preconditioner as Malfiet-Tjon bound state solver
     - Improves convergence for large systems
   - Supports complex scaling for resonance calculations

### Data Flow
- **Bound states**: System parameters → Channel coupling → Matrix elements → Eigenvalue problem → Binding energies and wavefunctions
- **Scattering**: System parameters + Initial state → Matrix assembly → Inhomogeneous equation → Scattering wavefunction and observables

## Important Implementation Details

### Fortran Integration
- Nuclear potentials implemented in Fortran for performance (AV18, AV14, Nijmegen models)
- Julia-Fortran interface via `ccall` and `Libdl.dlopen()` for dynamic library loading
- Automatic symbol resolution with name-mangling fallback patterns (`find_symbol()` function)
- Platform-specific dynamic library handling (`.dylib` on macOS, `.so` on Linux, `.dll` on Windows)

### Numerical Methods
- **Basis functions**: Laguerre polynomials for radial coordinates with regularization parameter
- **Integration**: Gauss-Legendre quadrature for angular components
- **Eigenvalue methods**: Two complementary approaches:
  - **Direct diagonalization**: Generalized eigenvalue problem `eigen(H, B)` finds all states
  - **Malfiet-Tjon iteration**: Reformulates as `λ(E)[c] = [E*B - T - V]⁻¹ * V*R * [c]`
- **Scattering solver**: Inhomogeneous equation solver with LU factorization or GMRES iterative method
- **Convergence**: Secant method iteration until `|λ(E) - 1| < tolerance`

### Faddeev Normalization (Critical for Truncated Model Space)
The framework uses **Faddeev normalization** which is essential for truncated model spaces:

**Two possible normalization schemes:**
1. **⟨Ψ|Ψ⟩ = 1**: Direct normalization of full wavefunction Ψ = (1 + Rxy)ψ₃
   - ❌ INCORRECT for truncated spaces
   - Rxy maps between Jacobi coordinates and is incomplete with finite lmax/λmax
   - Results in unconverged normalization

2. **3⟨Ψ|ψ₃⟩ = 1**: Faddeev normalization scheme
   - ✅ CORRECT for truncated spaces
   - Only involves ψ₃ which is fully defined within the truncated space
   - Converged even with finite basis size
   - Implementation: `|Ψ̄⟩ = |Ψ⟩/√(3⟨Ψ|ψ₃⟩)` ensures ⟨Ψ̄|Ψ̄⟩ = 1

**Channel Probability Calculation:**
- Computed from the full wavefunction Ψ̄: `P_channel = ⟨Ψ̄_ch|B|Ψ̄_ch⟩`
- Probabilities sum to ~98-99% (1-2% missing due to truncation)
- As lmax, λmax → ∞, sum approaches 100%
- All probabilities are positive (cross-terms can be negative, avoid using them)

### Channel Indexing
The framework uses sophisticated indexing schemes:
- Three-body channels: `|(l₁₂(s₁s₂)s₁₂)J₁₂, (λ₃s₃)J₃, J; (t₁t₂)T₁₂, t₃, T MT⟩`
- Matrix elements indexed by channel and coordinate grid points
- Pauli principle and parity constraints automatically enforced

## Working with the Code

### Adding New Potentials
1. Add Fortran implementation to `NNpot/` directory
2. Update makefile to include new source files
3. Add Julia wrapper function in `nuclear_potentials.jl`
4. Update `potential_type_to_lpot()` mapping

### Working with Three-Body Forces
- **UIX model**: Use `UIX.X12_matrix(α, grid)` to compute three-body force matrix
- **Matrix indexing**: Same as V and T matrices: `i = (iα-1)*grid.nx*grid.ny + (ix-1)*grid.ny + iy`
- **Angular momentum basis**: UIX functions implemented in Lagrange function basis, not Jacobi coordinate basis
- **Physical constants**: Uses PDG pion mass values with proper averaging formula
- **Delta functions**: Matrix elements include channel selection rules and coordinate constraints
- **Isospin phase convention**: The isospin phase factor is `(-1)^(T12_prime + 2*t1 + t2 + t3)` where T12_prime is from the **ket** (incoming channel), not the bra (outgoing channel). This must match the phase convention in `Gcoefficient.jl` line 91 for consistent angular momentum recoupling.

### Scattering Calculations
The framework supports scattering calculations with proper treatment of Coulomb interactions and complex scaling:

**Initial State Vector Construction:**
```julia
# 1. Compute two-body bound state (e.g., deuteron)
bound_energies, bound_wavefunctions = bound2b(grid, "AV18")
φ_d_matrix = ComplexF64.(bound_wavefunctions[1])  # Ground state with ³S₁ + ³D₁ components

# 2. Set up scattering parameters
E = 10.0   # Scattering energy (MeV)
z1z2 = 1.0 # Charge product (e.g., proton-deuteron)
θ = 0.0    # Complex scaling angle (radians, optional)

# 3. Compute initial state vector
φ = compute_initial_state_vector(grid, α, φ_d_matrix, E, z1z2, θ=θ)
```

**Key Physics:**
- Deuteron (J12=1) contains both ³S₁ (~96%) and ³D₁ (~4%) components
- Initial state populates ALL three-body channels coupling to deuteron
- Each channel uses its own λ for Coulomb function F_λ(ky)
- Channels with J12≠1 have zero wavefunction (no deuteron bound state exists)

**COULCC Library:**
- Provides regular (F) and irregular (G) Coulomb wavefunctions
- Interface: `coulcc(x, η, lmin; lmax=λmax, mode=4)` returns vector of F_λ values
- Complex scaling supported: evaluate at rotated coordinates x·exp(iθ)
- Optimized: single call returns all λ values from lmin to lmax

**Complex Scaling:**
- Used for resonance calculations and continuum discretization
- Matrices support complex scaling via backward rotation method
- `V_matrix_optimized_scaled(α, grid, potname, θ_deg=10.0)`
- `T_matrix_optimized(α, grid, θ_deg=10.0)` for kinetic energy
- Falls back to standard methods automatically when θ=0

### Modifying Calculations
- Channel configurations: Edit parameters in notebook initialization cells
- Mesh parameters: Adjust `nx`, `ny`, `xmax`, `ymax`, `alpha` for convergence
  - **Recommended for j2bmax=2.0**: nθ=12, nx=20, ny=20, xmax=16, ymax=16
  - Provides convergence within 0.2 keV while maintaining computational efficiency
  - For higher accuracy: increase to nx=25, ny=25 (converges within 1 keV)
- Potential models: Change `potname` variable to switch between models
- Three-body forces: Include UIX terms by adding X12_matrix to Hamiltonian construction

### Solver Selection and Performance
- **Direct method**: Use `ThreeBody_Bound()` when you need all eigenvalues or for initial exploration
- **Malfiet-Tjon method**: Use `malfiet_tjon_solve()` for ground state targeting, can be faster than direct method
- **Arnoldi optimization**: Recent performance improvements use optimized Arnoldi eigenvalue solver with adaptive convergence
- **Memory optimization**: Use smaller mesh sizes (20×20 instead of 30×30) and reduced λmax/lmax for memory-constrained systems
- **Performance consideration**: Malfiet-Tjon avoids expensive generalized eigenvalue problems
- **Initial guesses**: Use direct method results to inform Malfiet-Tjon starting energies for optimal convergence
- **Module conflicts**: Import MalflietTjon functions explicitly: `import .MalflietTjon: malfiet_tjon_solve`

### Debugging and Troubleshooting
- **Library loading**: Use `list_symbols(libpot)` to inspect available Fortran symbols
- **Symbol resolution**: `find_symbol()` handles platform-specific name mangling
- **Channel validation**: Check `α3b()` output for quantum number consistency and conservation laws
- **Matrix conditioning**: Monitor eigenvalue convergence and matrix conditioning in both solvers
- **Malfiet-Tjon convergence**: Use `verbose=true` to track λ eigenvalue behavior during iteration
- **Energy guesses**: For Malfiet-Tjon, start within ±1 MeV of expected ground state energy
- **Performance analysis**: Built-in timing analysis in main calculation routines
- **Notebook debugging**: Use `.ipynb` files for interactive problem investigation and method comparison
- **Symmetry checks**: Built-in rearrangement matrix transpose relationship validation (`Rxy_32 = Rxy_31^T`) with tolerance checking
- **Wave function analysis**: Channel probability contributions and normalization verification
  - Check that ⟨Ψ̄|ψ̄₃⟩ = 1/3 (Faddeev normalization)
  - Channel probabilities should sum to 98-99% (1-2% missing due to truncation is normal)
  - Negative channel probabilities indicate incorrect normalization scheme
- **Energy consistency**: Automated checks that ⟨ψ|H|ψ⟩ matches eigenvalue within tolerance
- **Truncation effects**: Monitor total probability sum - as lmax/λmax increase, sum approaches 100%

### Required Julia Packages
The project uses specific Julia packages that must be installed:
- **SphericalHarmonics**: For spherical harmonic calculations
- **WignerSymbols**: For angular momentum coupling coefficients
- **JSON**: For data serialization in notebooks
- **FastGaussQuadrature**: For numerical integration
- **Kronecker**: For tensor product operations
- **Revise**: For development workflow (hot reloading)

### Platform-Specific Notes
- **Dynamic libraries**: Build system automatically detects platform and uses appropriate extensions
- **macOS**: Creates `.dylib` files with `-dynamiclib -single_module -undefined dynamic_lookup`
- **Linux**: Creates `.so` files with `-shared` flag
- **Windows**: Creates `.dll` files with `-shared -Wl,--export-all-symbols`
- **Cross-platform compatibility**: Consistent interface across all platforms via Julia's `Libdl` module

### Error Handling and Common Issues
- **Library not found**: Ensure `make` completed successfully in `NNpot/` directory
- **Symbol resolution failures**: Check Fortran compilation and library loading with `list_symbols()`
- **Julia package issues**: Re-run `julia setup.jl` if missing dependencies
- **Module import conflicts**: Use explicit imports `import .MalflietTjon: function_name` instead of `using`
- **Malfiet-Tjon divergence**: Method fails with poor initial guesses - use direct method results as starting point
- **Numerical instabilities**: Adjust mesh parameters and check channel coupling convergence
- **Insufficient basis**: Small mesh sizes (< 20×20) may not capture three-body bound states

### Method Comparison Guidelines
- **Cross-validation**: Always compare direct and Malfiet-Tjon results for the same system
- **Energy convergence**: Both methods should agree to at least 6 decimal places for ground state
- **Computational trade-offs**: Malfiet-Tjon faster for single state, direct method gives complete spectrum
- **Physical insight**: Malfiet-Tjon iteration demonstrates bound state formation mechanism via λ → 1

### Advanced Features and Optimizations
- **Arnoldi eigenvalue solver**: Adaptive convergence with early termination for computational efficiency
- **Caching system**: Automated caching of expensive Wigner coefficients and angular momentum calculations
- **Memory optimization**: Progressive mesh refinement and sparse matrix representations where applicable
- **Cross-platform symbol resolution**: Automatic handling of Fortran name-mangling across different platforms
- **Comprehensive validation framework**: Built-in physics consistency checks and numerical stability monitoring
- **Rearrangement matrix validation**: Automatic verification that `Rxy_32 = Rxy_31^T` (transpose relationship) as required by Faddeev coordinate transformation symmetry
- **Performance profiling**: Integrated timing analysis for identifying computational bottlenecks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jinleiphys) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
