## medical-data-visualization-3d

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Medical-Data-Visualization-3D is a C++ medical imaging application that provides 3D visualization and analysis of medical data including DICOM, MRI, and CT scans. The application uses ITK (Insight Segmentation and Registration Toolkit) and VTK (Visualization Toolkit) libraries to process and render medical images.

## Build and Development Commands

### Building the Project
```bash
mkdir build && cd build
cmake ../src
# On Windows: Open the generated .sln file in Visual Studio and build
# On Linux/Mac: make
```

### Prerequisites
- Visual Studio >= 2015 (Windows)
- CMake
- VTK library (https://www.youtube.com/watch?v=IgvbhyDh8r0)
- ITK library (https://www.youtube.com/watch?v=vZOMu5YSfoI)
- Run as Administrator (Windows)

### Executable Location
After building, the executable is located in `/bin/debug/` folder.

## Architecture Overview

### Core Application Structure
- **PROJECT.cxx**: Main entry point with console menu system offering 5 medical visualization operations
- **Header-only modules**: All functionality is implemented in header files (.h) that contain both declarations and implementations

### Key Functional Modules

1. **Volume Rendering** (`vtkVolumeRenderer.h`)
   - Brain data volume rendering with GPU ray casting
   - Supports both DICOM directories (-DIR) and MetaImage files (-FILE)
   - Complex color transfer functions for anatomical visualization
   - Uses vtkGPUVolumeRayCastMapper for performance

2. **DICOM Series Viewer** (`readDICOMSeries.h`)
   - Interactive slice-by-slice viewing of DICOM series
   - Custom vtkInteractorStyleImage with keyboard/mouse wheel navigation
   - Real-time slice number display and usage instructions

3. **Medical Segmentation** (`ConfidenceConnected3D.h`)
   - ITK-based confidence connected segmentation for brain white matter
   - ITK-to-VTK pipeline conversion for visualization
   - Predefined seed points for brain segmentation
   - Combines image processing (ITK) with 3D rendering (VTK)

4. **CT Bone Rendering** (`FixedPointVolumeRayCastMapperCT.h`)
   - Specialized volume rendering for CT bone data
   - Fixed-point ray casting for bone visualization

5. **Watershed Segmentation** (`itkWaterShedCode.h`)
   - ITK watershed algorithm implementation
   - Advanced image segmentation capabilities

### Technology Stack
- **C++**: Core implementation language
- **ITK**: Medical image processing and analysis
- **VTK**: 3D visualization and rendering
- **CMake**: Cross-platform build system
- **Windows-specific**: Uses windows.h, dos.h for system operations

### Data Flow Pattern
The application follows a consistent pattern:
1. User selects operation from console menu
2. Input file/directory path specification
3. ITK processing (for segmentation) or direct VTK loading
4. VTK pipeline setup with volume properties and transfer functions
5. Interactive 3D rendering window with trackball camera controls

### Key Implementation Notes
- All visualization uses GPU-accelerated volume ray casting when available
- Complex predefined color transfer functions for anatomical accuracy
- ITK-VTK pipeline integration using specialized connector templates
- Custom interactor styles for medical image navigation
- Platform-specific code includes Windows dependencies

### File Structure Conventions
- Main executable: PROJECT.cxx
- Functional modules: Descriptive header names ending in .h
- Build configuration: CMakeLists.txt in src/ directory
- All implementation in headers (no separate .cpp files for modules)

---
> Source: [ashahrior/Medical-Data-Visualization-3D](https://github.com/ashahrior/Medical-Data-Visualization-3D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
