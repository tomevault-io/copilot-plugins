## architecture-smart-calculate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an architectural calculation tool (架构计算工具) designed to process structural engineering data from SATWE output files. The application uses AI to extract and process structural column data from text files and perform engineering calculations.

## Key Commands

### Running the Application
```bash
# Run the main application
python src/main.py

# Run using virtual environment
.venv\Scripts\activate
python src/main.py
```

### Building the Executable
```bash
# Build using PyInstaller (creates executable in dist/)
pyinstaller 架构计算工具.spec

# Alternative build command
pyinstaller --onefile --name "架构计算工具" src/main.py
```

### Installing Dependencies
```bash
# Install from requirements.txt
pip install -r requirements.txt

# Create virtual environment
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

## Architecture

### Core Components

1. **main.py** (`src/main.py`): The main application file containing:
   - `load_data_ai()`: Uses Zhipu AI client to extract structural data from text files
   - `load_file()`: File selection dialog with size validation (200KB limit)
   - `calculate_result()`: Performs structural engineering calculations

2. **Data Processing Flow**:
   - Reads SATWE output files (Chinese structural engineering software)
   - Uses AI (Zhipu AI GLM-4.6 model) to parse and extract column data
   - Converts extracted data to structured pandas DataFrame
   - Performs structural stability calculations

3. **Key Dependencies**:
   - `zai-sdk`: Zhipu AI SDK for LLM integration
   - `pandas`: Data manipulation and analysis
   - `tkinter`: GUI for file selection
   - `numpy`: Numerical calculations
   - `pyinstaller`: For creating standalone executable

### Data Format
- Input: SATWE output files (`.OUT` format) containing structural analysis results
- Output: Pandas DataFrame with columns:
  - N-C (column number), Node-i, Node-j, DL (length), Angle
  - Internal forces: iCase, Shear-X, Shear-Y, Axial, Mx-Btm, My-Btm, Mx-Top, My-Top

### Engineering Calculations
The tool performs stability calculations for structural columns including:
- Axial force calculations
- Bending moment analysis
- Stability coefficient calculations
- Combined stress verification

## Development Notes

### API Configuration
The Zhipu AI API key is hardcoded in `main.py` (line 17). For production use, this should be moved to environment variables or configuration files.

### File Size Limitation
Files are limited to 200KB for AI processing to avoid token limits and ensure reliable parsing.

### Encoding
The application processes Chinese text files from SATWE, ensure proper encoding when modifying file handling.

### Testing
No test framework is currently set up. Consider adding pytest for unit testing the calculation functions.

---
> Source: [daijin0611/architecture-smart-calculate](https://github.com/daijin0611/architecture-smart-calculate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
