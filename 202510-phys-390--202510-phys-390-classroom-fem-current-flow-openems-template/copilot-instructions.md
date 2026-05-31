## 202510-phys-390-classroom-fem-current-flow-openems-template

> - OpenEMS FDTD electromagnetic simulator

# OpenEMS + ElmerFEM Development Container - Project Summary

## Branch Structure

**main branch:**
- OpenEMS FDTD electromagnetic simulator
- High-frequency signal integrity analysis
- VTK field visualization for ParaView

**ElmerFEM branch:**
- Adds ElmerFEM finite element solver
- DC/low-frequency current flow analysis
- Power distribution network analysis
- Combines both simulation tools

## What's Working - OpenEMS (main + ElmerFEM branches)

### Docker Environment
- ✅ Dockerfile builds successfully (~20 min on Apple Silicon)
- ✅ All dependencies installed (OpenEMS, CSXCAD, VNC, etc.)
- ✅ x86_64 emulation working on ARM
- ✅ Image size: ~2-3 GB
- ✅ Single `setup_gui.sh` script starts everything

### VNC Desktop
- ✅ XFCE4 desktop starts correctly
- ✅ Accessible via http://localhost:6080/vnc.html
- ✅ Password: `openems`
- ✅ Matplotlib plots display in VNC window
- ✅ Ctrl+C gracefully shuts down

### OpenEMS Installation
- ✅ CSXCAD v0.6.3 compiled and installed
- ✅ OpenEMS v0.0.36 compiled and installed
- ✅ Python bindings working
- ✅ All imports successful (CSXCAD, openEMS, numpy, matplotlib, h5py)
- ✅ Official tutorials working perfectly

### OpenEMS Field Visualization - **WORKING!**
- ✅ **plane_wave_simple.py** - Clean electromagnetic field propagation
  - Plane wave excitation (no port configuration issues)
  - Shows E-field and H-field propagation
  - Demonstrates wave interaction with dielectric
  - Exports VTK files for ParaView visualization
  - ~1-2 minute simulation, ~2000 VTK files

- ✅ **simple_field_dump.py** - Minimal field visualization example

- ✅ **microstrip_with_vtk.py** - Advanced microstrip with fields
  - Has port configuration warnings but VTK dumps work independently
  - Shows fields around PCB trace

### Critical Fix: OpenEMS Coordinate System
**Problem discovered:** All geometry in OpenEMS (materials, excitations, dumps, ports) must use **mesh coordinates**, NOT SI units!

**Wrong:**
```python
mesh.SetDeltaUnit(1e-3)  # mm to meters
mesh.AddLine('x', np.linspace(0, 40, 21))  # 0 to 40 mm
dump.AddBox(start=[0, 0, 0], stop=[40*unit, 40*unit, 60*unit])  # ✗ SI units
```

**Correct:**
```python
mesh.SetDeltaUnit(1e-3)  # mm to meters
mesh.AddLine('x', np.linspace(0, 40, 21))  # 0 to 40 mm
dump.AddBox(start=[0, 0, 0], stop=[40, 40, 60])  # ✓ Mesh coordinates
```

This fix was applied to all geometry definitions: materials, excitations, dumps, and ports.

### ParaView Visualization
- ✅ VTK export working correctly
- ✅ 3D electromagnetic field data
- ✅ Time-evolution animation of wave propagation
- ✅ PARAVIEW_GUIDE.md created with complete instructions

## What's Working - ElmerFEM (ElmerFEM branch only)

### ElmerFEM Installation
- ✅ Built from source (~15-30 min additional build time)
- ✅ ElmerSolver, ElmerGrid, ElmerGUI installed
- ✅ MUMPS and Hypre solvers included
- ✅ Gmsh mesh generator (Python API)
- ✅ Python packages: gmsh, meshio, pygmsh, pyvista

### ElmerFEM Examples - **WORKING!**
- ✅ **simple_resistor.py** - Basic test (rectangle with voltage applied)
  - Validates ElmerFEM installation
  - Simple 2D conductor
  - Direct solver, guaranteed convergence

- ✅ **tapered_working.py** - Tapered PCB trace with current density
  - 2D copper trace: 4mm → 1mm width over 15mm length
  - DC current flow analysis
  - Shows current crowding in narrow section
  - Exports: Potential, Electric Field, Current Density, Joule Heating
  - VTK output for ParaView visualization

### Critical Fix: ElmerFEM Field Export
**Problem:** ElmerFEM calculates fields but doesn't automatically export them to VTU files.

**Solution:** Use `Exported Variable` in solver configuration:
```
Solver 1
  Calculate Electric Field = True
  Calculate Current Density = True

  ! Force export to VTU
  Exported Variable 1 = -dofs 3 Electric Field
  Exported Variable 2 = -dofs 3 Current Density
End
```

Also must use **Direct Solver (UMFPack)** instead of iterative solvers for complex geometries to ensure convergence.

### ElmerFEM vs OpenEMS

**Use ElmerFEM for:**
- DC or low-frequency (<1 MHz) analysis
- Resistive voltage drops (IR drop)
- Power distribution networks
- Current density in conductors
- Thermal-electrical coupling
- Static field problems

**Use OpenEMS for:**
- High-frequency (>10 MHz) analysis
- Transmission lines, S-parameters
- Signal integrity, impedance
- Antennas, RF circuits
- Wave propagation
- Time-domain electromagnetics

## Student Workflow

### Start GUI (both branches)
```bash
bash setup_gui.sh
# Opens: http://localhost:6080/vnc.html
# Password: openems
```

### OpenEMS Examples
```bash
cd /workspace/examples
python3 plane_wave_simple.py  # Clean field visualization, no port errors
python3 simple_field_dump.py   # Minimal example
```

### ElmerFEM Examples (ElmerFEM branch)
```bash
cd /workspace/elmerfem-examples
python3 simple_resistor.py     # Basic validation test
python3 tapered_working.py     # Tapered trace with current density
```

### Official OpenEMS Tutorials
```bash
cd /workspace/Tutorials
python3 Rect_Waveguide.py
# Plots appear in VNC desktop
```

## Documentation

### Main Documentation
- ✅ **README.md** - Quick start guide
- ✅ **PARAVIEW_GUIDE.md** - Complete ParaView visualization guide
- ✅ **Tutorials/README.md** - Official OpenEMS tutorials guide
- ✅ **examples/README.md** - OpenEMS custom examples guide
- ✅ **elmerfem-examples/README.md** - ElmerFEM examples guide (ElmerFEM branch)

### Technical Details
- ✅ **DEPLOYMENT.md** - Full vision for teaching environment
- ✅ **CLAUDE.md** - This file, project summary and status

## Known Issues and Limitations

### OpenEMS Port Configuration
- Lumped ports have configuration complexity
- Microstrip ports require careful mesh alignment
- Solution: Use proven examples from official tutorials
- Field visualization examples avoid ports entirely

### ElmerFEM Convergence
- Iterative solvers (BiCGStab) can fail on complex geometry
- Solution: Use Direct Solver (UMFPack) for guaranteed convergence
- 3D geometries with extreme aspect ratios problematic
- Solution: Use 2D models or improve geometry ratios

### Build Time
- **OpenEMS only:** ~20 minutes
- **OpenEMS + ElmerFEM:** ~50-60 minutes (ElmerFEM builds from source)
- One-time cost, image can be reused

## File Structure

```
.devcontainer/
├── Dockerfile              # OpenEMS or OpenEMS+ElmerFEM
├── devcontainer.json       # VS Code config
└── post-create.sh          # Post-creation setup

Root:
├── setup_gui.sh            # ONE script to start everything
├── test_environment.py     # Verify installation (ElmerFEM branch)
├── test_openems.py         # Verify OpenEMS (main branch)
├── README.md               # User guide
├── PARAVIEW_GUIDE.md       # ParaView instructions
├── DEPLOYMENT.md           # Full vision
└── CLAUDE.md               # This file

examples/ (OpenEMS):
├── README.md
├── plane_wave_simple.py    # ✓ Working - no ports, clean fields
├── simple_field_dump.py    # ✓ Working - minimal example
└── microstrip_with_vtk.py  # ✓ Working - advanced, port warnings

elmerfem-examples/ (ElmerFEM branch only):
├── README.md
├── simple_resistor.py      # ✓ Working - basic test
├── tapered_working.py      # ✓ Working - current density
└── inspect_vtu.py          # VTU file inspection tool

Tutorials/ (Official OpenEMS):
├── README.md
└── *.py                    # ✓ All working with GUI
```

## Success Criteria - ACHIEVED

### OpenEMS
- [x] Docker environment builds and runs
- [x] VNC desktop accessible via browser
- [x] OpenEMS simulations run successfully
- [x] Electromagnetic field visualization working
- [x] VTK export to ParaView working
- [x] Documentation complete
- [x] Single-script startup (setup_gui.sh)
- [x] Official tutorials working with GUI plots

### ElmerFEM (ElmerFEM branch)
- [x] ElmerFEM builds and installs
- [x] Gmsh mesh generation working
- [x] FEM solver converges successfully
- [x] Current density calculation working
- [x] VTK export with all fields working
- [x] ParaView visualization of results
- [x] Working examples created
- [x] Documentation complete

## Key Lessons Learned

1. **OpenEMS coordinate system:** Everything must use mesh coordinates, not SI units
2. **ElmerFEM field export:** Must explicitly declare `Exported Variable` to save computed fields
3. **ElmerFEM convergence:** Direct solvers are more robust than iterative for complex geometry
4. **VTK inspection:** Created Python tool to debug VTU file contents
5. **Port complexity:** Field visualization examples work better without ports for teaching
6. **Build caching:** Docker BuildKit cache can cause issues; use `docker builder prune -a -f`
7. **Single script simplicity:** Students prefer one script (setup_gui.sh) over multiple steps

## Next Steps (Optional Enhancements)

### Potential Improvements
- [ ] 3D ElmerFEM examples with better aspect ratios
- [ ] Thermal-electrical coupling in ElmerFEM
- [ ] Multi-layer PCB power distribution analysis
- [ ] Fix OpenEMS microstrip port configuration for S-parameter analysis
- [ ] Add more field visualization examples
- [ ] Create hybrid examples (ElmerFEM for DC, OpenEMS for AC)
- [ ] Add mesh refinement tutorials
- [ ] Create assignment templates for students

### Deployment Options
- [ ] GitHub Codespaces integration (if needed)
- [ ] Pre-built Docker images on registry (to skip build time)
- [ ] Jupyter notebook interface
- [ ] GitHub Classroom integration

## Conclusion

**Status: Production Ready**

Both OpenEMS and ElmerFEM environments are fully functional with:
- Working examples for electromagnetic field simulation (FDTD)
- Working examples for current flow analysis (FEM)
- Complete ParaView visualization workflow
- Comprehensive documentation
- Simple single-script startup
- Validated on Apple Silicon via x86_64 emulation

Students can now:
1. Start the environment with one command
2. Run proven simulation examples
3. Visualize results in ParaView
4. Learn both high-frequency (OpenEMS) and DC/low-frequency (ElmerFEM) analysis

The environment successfully demonstrates PCB electromagnetic simulation suitable for teaching.

---
> Source: [202510-PHYS-390/202510-phys-390-classroom-fem-current-flow-openems-template](https://github.com/202510-PHYS-390/202510-phys-390-classroom-fem-current-flow-openems-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
