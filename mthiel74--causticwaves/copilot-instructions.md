## causticwaves

> Generates band-limited height fields using FFT-based synthesis.

# CausticWaves Project Guide

## Project Overview

CausticWaves is a Python-based simulation of optical caustics - the beautiful "dancing light" patterns seen at the bottom of swimming pools. It models the interaction of sunlight with a dynamic water surface to compute realistic intensity distributions on the pool floor.

## Technical Approach

### Core Technologies

1. **Spectral Synthesis (Tessendorf Waves)**
   - Uses Fast Fourier Transforms (FFT) on a band-limited spectrum
   - Preferred over PDE-based fluid simulations for aesthetic realism
   - Produces realistic ripple patterns optimized for optical rendering
   - Implements Phillips/Tessendorf spectrum approach common in oceanography graphics

2. **Geometric Optics**
   - Ray tracing from directional light source (Sun)
   - Snell's Law refraction at air-water interface
   - Ray-plane intersection with pool bottom
   - Fresnel effects using Schlick's approximation

3. **Accumulation Buffer**
   - High-resolution grid histogram for ray hit accumulation
   - Creates characteristic sharp bright caustic lines
   - Tonemapping for high dynamic range handling

## Code Structure

### Main Files

- `simulation.py`: Standalone script with full implementation
- `CausticWaves.ipynb`: Jupyter notebook version for interactive exploration
- `Output/`: Generated visualizations (GIF, MP4, PNG)

### Key Components

#### 1. `SpectralRipples` Class (lines 132-229)

Generates band-limited height fields using FFT-based synthesis.

**Key Methods:**
- `__init__()`: Initializes wave spectrum with dispersion relation ω² = gk tanh(kH)
- `height(t)`: Returns spatial height field at time t using analytic time evolution

**Important Parameters:**
- `Nx, Ny`: Grid resolution (default: 256x256)
- `Lx, Ly`: Physical dimensions in meters (default: 2.0m x 2.0m)
- `depth`: Water depth in meters (default: 1.2m)
- `target_rms`: Root-mean-square wave height (default: 0.004m = 4mm)
- `lambda0`: Dominant wavelength (default: 0.15m)
- `bandwidth`: Frequency spread controlling wave chaos

#### 2. `refract_air_to_water()` (lines 66-110)

Vectorized Snell's Law implementation for 3D refraction.

**Returns:**
- `tx, ty, tz`: Transmitted ray vector components
- `cosi`: Cosine of incident angle (used for Fresnel calculation)

**Physics:**
- Vector form: T = η·I + (η·cos(θᵢ) - √(1 - η²·sin²(θᵢ)))·N
- Handles total internal reflection (though rare for air→water)

#### 3. `caustics_frame()` (lines 247-342)

Main rendering pipeline for a single timestep.

**Process:**
1. Generate surface H(x,y,t) from spectral ripples
2. Compute surface normals via finite differences
3. Refract sun rays through normals
4. Intersect transmitted rays with bottom plane
5. Accumulate ray hits into 2D histogram
6. Apply blur to simulate finite sun angular size
7. Tonemapping: normalize → log → gamma correction

**Key Parameters:**
- `bottom_res`: Output image resolution (default: 700x700)
- `sun_elev_deg`: Sun elevation angle (0=horizon, 90=zenith)
- `sun_az_deg`: Sun azimuth angle
- `exposure`: Exposure multiplier for tonemapping (default: 60.0)
- `gamma`: Gamma correction value (default: 0.7)
- `blur_sigma_px`: Gaussian blur sigma in pixels

#### 4. Helper Functions

- `sun_direction()` (lines 43-63): Converts spherical coordinates to 3D direction vector
- `fresnel_transmission_schlick()` (lines 113-125): Approximates light transmission through interface
- `gaussian_blur_fft()` (lines 236-244): Fallback FFT-based blur if scipy unavailable

## Physics & Mathematics

### Dispersion Relation

```
ω² = g·k·tanh(k·H)
```

Where:
- ω = angular frequency
- g = gravitational acceleration (9.81 m/s²)
- k = wavenumber (2π/λ)
- H = water depth

This ensures realistic wave propagation speeds based on wavelength and depth.

### Hermitian Symmetry

The FFT implementation enforces h₀(-k) = h₀*(k) to guarantee real-valued spatial fields. This is critical for preventing imaginary components in the height field.

### Ray Weighting

Ray intensity at the bottom is weighted by:
```
w = T_fresnel · cos(θᵢ) / nz
```

Where:
- T_fresnel: Transmission coefficient (1 - reflectance)
- cos(θᵢ): Lambertian term (projected area)
- 1/nz: Flux density correction

## Dependencies

**Required:**
- `numpy`: Core array operations and FFT
- `matplotlib`: Visualization and animation

**Optional:**
- `scipy`: High-quality Gaussian blur (falls back to FFT blur if missing)

## Common Modifications

### Adjusting Wave Characteristics

**Make waves larger:**
```python
target_rms = 0.008  # Double the wave height
```

**Make waves more chaotic:**
```python
bandwidth = 0.12  # Wider frequency spread
```

**Change dominant wavelength:**
```python
lambda0 = 0.20  # Larger ripples
```

### Lighting Adjustments

**Change sun angle:**
```python
sun_elev_deg = 45.0  # Lower sun, longer shadows
sun_az_deg = 0.0     # Sun from different direction
```

**Adjust brightness:**
```python
exposure = 100.0  # Brighter
gamma = 0.6       # More contrast
```

### Performance Tuning

**Faster rendering (lower quality):**
```python
Nx = Ny = 128        # Lower simulation resolution
bottom_res = 500     # Lower output resolution
blur_sigma_px = 0.5  # Less blur
```

**Higher quality (slower):**
```python
Nx = Ny = 512
bottom_res = 1000
blur_sigma_px = 1.5
```

## Running the Simulation

### Standalone Script
```bash
python simulation.py
```

### Jupyter Notebook
```bash
jupyter notebook CausticWaves.ipynb
```

## Output Formats

The simulation can export to:
- **Live animation**: Real-time matplotlib window
- **GIF**: For web sharing (see `Output/caustics_simulation.gif`)
- **MP4**: For video playback (see `Output/caustics_simulation.mp4`)
- **PNG**: Static frames (see `Output/caustics_preview.png`)

## Technical Notes

### Why Spectral Synthesis Over PDEs?

The project uses FFT-based spectral synthesis rather than solving Shallow Water Equations because:
1. Better aesthetic control over wave spectrum
2. Exact dispersion relations
3. Faster computation via FFT
4. Seamless tiling (periodic boundary conditions)
5. More "pool-like" appearance

### Coordinate System

- **x, y**: Horizontal plane (pool surface)
- **z**: Vertical axis (positive = up, negative = down)
- **z = h(x,y,t)**: Water surface elevation
- **z = -H**: Pool bottom (constant depth H)

### Wrapping Behavior

The simulation uses periodic boundary conditions (`wrap=True`) so rays exiting one edge re-enter from the opposite edge. This creates seamless tiling for animation loops.

## Potential Extensions

1. **Add color dispersion**: Simulate chromatic aberration by varying n_water with wavelength
2. **Interactive controls**: Add sliders for real-time parameter adjustment
3. **3D visualization**: Render both surface and bottom simultaneously
4. **Multiple light sources**: Combine caustics from different angles
5. **Particle injection**: Track floating objects casting shadows
6. **GPU acceleration**: Port to CUDA/OpenCL for real-time high-resolution rendering

## Troubleshooting

**Animation is slow:**
- Reduce `bottom_res` or `Nx, Ny`
- Decrease `frames` count
- Disable blur (`blur_sigma_px=None`)

**Caustics look wrong:**
- Check sun angle isn't too shallow (< 30°)
- Verify depth > 0 and reasonable (0.5-2.0m)
- Ensure wave_rms isn't too large (> 0.01m causes unrealistic refraction)

**Dark or washed out:**
- Adjust `exposure` and `gamma` parameters
- Check Fresnel weights aren't being clipped

## Scientific Background

This simulation implements principles from:
- **Computer Graphics**: Tessendorf waves (SIGGRAPH 2001)
- **Optics**: Snell's Law, Fresnel equations
- **Oceanography**: Phillips spectrum, dispersion relations
- **Rendering**: Accumulation buffers, tonemapping

The approach balances physical accuracy with computational efficiency and visual appeal.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mthiel74) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
