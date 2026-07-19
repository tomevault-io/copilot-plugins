## sfrtg

> Generates geometric patterns of spheres (vertices) inside a designated Target ROI (GTV/PTV).

# SFRT Planning & Visualization Tool

This repository contains a comprehensive suite of Python tools for Spatially Fractionated Radiation Therapy (SFRT) planning, analysis, and visualization. It features a modern GUI for interacting with DICOM-RT data, generating lattice structures (Vertices/Spheres) within target volumes, and optimizing parameters.

## Core Features

### 1. DICOM-RT Visualization (GUI)
A high-performance desktop application (`main_gui.py`) built with **PySide6**, **PyQtGraph**, and **VTK**.
*   **Data Import:** Asynchronous loading of CT Image Series, RTSTRUCT, and RTDOSE files from a directory.
*   **Patient Explorer:** Hierarchical tree view of loaded patient data and series.
*   **2D Visualization:** Fast axial CT view with overlay for ROIs (Contours) and Isodose Lines.
*   **3D Visualization:** Interactive 3D surface rendering of ROIs using VTK.
*   **ROI Management:** Toggle visibility, select specific ROIs for processing.
*   **Dose Analysis:** Color Wash and Isodose Line overlays (95%, 80%, 50%, 30%).

### 2. SFRT Sphere Generation (`SFRTG.py`)
Generates geometric patterns of spheres (vertices) inside a designated Target ROI (GTV/PTV).
*   **Algorithms:** 
    *   **CVT:** Centroidal Voronoi Tessellation (Uniform distribution).
    *   **HCP:** Hexagonal Closest Packed (Dense packing).
    *   **SCP:** Simple Cubic Packing.
    *   **AC:** Alternating Cubic.
*   **Parameters:** Configurable Sphere Radius (`R`) and Spacing (`D`).
*   **Clipping:** Option to clip spheres to the ROI boundary.

### 3. Parameter Optimization (`SFRTO.py`)
Adaptive strategy to select optimal `R` and `D` values based on the target ROI's geometry (volume, shape).
*   **Auto-Calculation:** Analyzes GTV volume to determine inward margins and packing density.
*   **Monte Carlo Optimization:** Finds the best lattice offset and rotation to maximize sphere count within the target.

## System Architecture

The project follows a modular **Model-View-Controller (MVC)** design:

*   **Model (`DICOMLoader`, `dicomP`):** Handles DICOM file I/O using `pydicom`. `dicomP.py` parses complex RTSTRUCT sequences. Data is stored in efficient numpy arrays.
*   **View (`ImageRenderer`, `main_gui`):** 
    *   **2D:** `PyQtGraph` for real-time slicing and contour drawing.
    *   **3D:** `VTK` for hardware-accelerated surface rendering of anatomical structures.
*   **Controller (`AppController`):** Manages application state, thread synchronization (via `QThread`), and interaction between the UI and data models.

## Dependencies

*   **Python:** >= 3.12
*   **GUI:** `PySide6`, `pyqtgraph`
*   **Visualization:** `vtk`, `matplotlib` (legacy)
*   **Data Processing:** `numpy`, `scipy`, `pydicom`, `shapely`, `simpleitk`, `scikit-image`
*   **DICOM RT:** `dicompyler-core`

## Installation

Ensure you have a Python 3.12 environment. Install dependencies:

```bash
pip install PySide6 pyqtgraph vtk pydicom numpy scipy shapely simpleitk scikit-image dicompyler-core
```

## Usage

### Graphical User Interface (Recommended)

Run the main application:

```bash
python main_gui.py [optional_path_to_dicom_folder]
```

**Workflow:**
1.  **Import:** Click "Import CT Folder" to load a patient directory containing CT and RS files.
2.  **Visualize:** Use the slider to navigate slices. Toggle "3D View" for a volumetric perspective.
3.  **SFRT Generation:**
    *   Select the **Target ROI** (e.g., GTV) from the dropdown.
    *   Choose an **Algorithm** (or "Auto" for optimization).
    *   Set **R** (Radius) and **D** (Spacing) parameters.
    *   Click **Create**.
    *   Review statistics (Volume, Sphere Count, Ratio) in the output box.
    *   The new ROI (e.g., `GTV_CVT`) is automatically added to the viewer.

### Command Line Tools

*   **Parser:** `python dicomP.py <rtstruct.dcm>`
*   **Generator:** `python SFRTG.py <plan.roi> <TargetROI>` (Legacy Pinnacle format support)
*   **Optimizer:** `python SFRTO.py <plan.roi> <TargetROI>`

## Project Structure

*   `main_gui.py`: Main entry point for the GUI application.
*   `dicomP.py`: DICOM-RT Structure Set parsing library.
*   `SFRTG.py`: Core logic for sphere packing and geometry generation.
*   `SFRTO.py`: Optimization algorithms for SFRT parameters.
*   `GEMINI.md`: Project documentation.

---
> Source: [sparkpoints/SFRTG](https://github.com/sparkpoints/SFRTG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
