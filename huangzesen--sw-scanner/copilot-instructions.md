## sw-scanner

> Solar wind scanner using Jensen-Shannon divergence to detect non-Gaussian structures in solar wind time series data. Use when analyzing PSP (Parker Solar Probe) magnetic field data, solar wind turbulence, or any time series where you need to detect distributional changes.


# sw-scanner

Solar Wind Scanner — detecting non-Gaussian structures in solar wind data using Jensen-Shannon divergence.

## When to Use

- Analyzing Parker Solar Probe (PSP) magnetic field data
- Detecting distributional changes in time series
- Solar wind turbulence analysis
- Identifying non-Gaussian structures in plasma data

## Quick Start

```python
from sw_scanner import SolarWindScanner
import numpy as np

# Prepare data
Btot = np.array([...])  # magnetic field magnitude
Dist_au = np.array([...])  # distance in AU

# Run scan
results = SolarWindScanner(
    Btot, Dist_au,
    settings={'wins': np.array([np.timedelta64(30, 's')]),
              'step': np.timedelta64(1, 's'),
              'normality_mode': 'divergence'},
    Ncores=8,
)
```

## API Reference

### Main Functions

- `SolarWindScanner(Btot, Dist_au, settings, ...)` — 1-D scanner (main entry point)
- `js_distance(a, b)` — Jensen-Shannon distance between two distributions
- `round_up_to_minute(ts)` / `round_down_to_minute(ts)` — timestamp rounding utilities
- `f(x, a, b)` — linear function for fitting

### Modules

- `sw_scanner.scanner` — main 1-D scanner
- `sw_scanner.ndims` — multi-dimensional scanner
- `sw_scanner.reduced` — reduced/adaptive scanner
- `sw_scanner.filters` — Hampel filter (requires numba)
- `sw_scanner.lib` — utility functions

## Dependencies

Required: numpy, scipy, tqdm, numba, pandas

## Installation

```bash
pip install sw-scanner
# or from source:
pip install -e /path/to/sw_scanner_package
```

## Source

`/Users/huangzesen/work/GitHub/sw_scanner/`

---
> Source: [huangzesen/sw_scanner](https://github.com/huangzesen/sw_scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
