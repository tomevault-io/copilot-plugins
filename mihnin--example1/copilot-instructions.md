## example1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a data analysis example project (`example1`) containing sales data analysis code. The project demonstrates:

- Data analysis workflow using pandas and matplotlib
- Processing Excel files with sales data
- Statistical analysis and visualization of time series sales data
- Bilingual documentation (Russian/English)

## Repository Structure

```
C:\dev\example1\
├── docs/                    # Documentation and data files
│   ├── sample_sales_data.xlsx   # Sample Excel data file
│   ├── пример1.ipynb           # Jupyter notebook (Russian)
│   └── пример1.py              # Python script converted from notebook
├── tests/                   # Comprehensive test suite
├── .github/workflows/       # CI/CD pipeline configuration
├── sample_sales_data.xlsx   # Sample data (also in root for quick access)
├── streamlit_app.py         # Main application entry point
├── requirements.txt         # Python dependencies
├── Dockerfile              # Container configuration
├── README.md               # Basic project description
└── LICENSE                # Project license
```

## Key Files

- `streamlit_app.py`: **Main Streamlit web application** for interactive data analysis with file upload (fully self-contained)
- `docs/sample_sales_data.xlsx`: Sample Excel file for local development (not included in production Docker)
- `docs/пример1.py`: Original data analysis script (development reference only)
- `docs/пример1.ipynb`: Original Jupyter notebook (generated from Google Colab)
- `requirements.txt`: Production dependencies (cleaned from Windows-specific packages)
- `tests/`: Comprehensive test suite with unit, integration, and quality tests
- `pytest.ini`: Pytest configuration file
- `tests/conftest.py`: Shared test fixtures and configuration
- `Dockerfile`: Docker containerization configuration
- `.dockerignore`: Files excluded from Docker build context
- `.github/workflows/ci-cd.yml`: GitHub Actions CI/CD pipeline

## Development Environment

This project is developed in:
- **Python 3.13+**: Data analysis using pandas and matplotlib
- **Virtual Environment**: Use `python -m venv venv` and `.\venv\Scripts\activate` (Windows)
- **Jupyter Notebooks**: Interactive development environment (Google Colab origin)
- **Excel**: Data storage format (.xlsx files)

## Data Analysis Workflow

The main analysis script (`пример1.py`) performs:

1. **Data Loading**: Loads Excel data using pandas
2. **Data Overview**: Displays data types and basic information
3. **Statistical Analysis**: Calculates descriptive statistics for numerical columns
4. **Data Analysis**: Groups data, calculates totals and averages, identifies trends
5. **Visualization**: Creates time series plots and bar charts using matplotlib
6. **Reporting**: Generates detailed analysis report in both Russian and English

## Required Dependencies

Install dependencies: `pip install -r requirements.txt`

Or install core packages manually: `pip install streamlit pandas openpyxl matplotlib seaborn`

Core libraries:
- `pandas`: Data manipulation and analysis
- `matplotlib`: Data visualization and plotting
- `openpyxl`: Excel file reading/writing
- `seaborn`: Statistical data visualization (optional enhancement)
- `streamlit`: Web app framework for interactive dashboards
- Standard datetime handling for time series analysis

## Development Commands

### Environment Setup
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment (Windows)
.\venv\Scripts\activate

# Install dependencies from requirements file
pip install -r requirements.txt

# Or install core packages manually
pip install streamlit pandas openpyxl matplotlib seaborn
```

### Running the Analysis
```bash
# Run the main analysis script (requires fixing file path)
python docs/пример1.py

# Start Streamlit web application
python -m streamlit run streamlit_app.py
# App will be available at: http://localhost:8501
```

### Docker Commands
```bash
# Build Docker image locally
docker build -t sales-analytics-app .

# Run Docker container
docker run -p 8501:8501 sales-analytics-app

# Run with volume mount for data persistence
docker run -p 8501:8501 -v $(pwd)/data:/app/data sales-analytics-app

# Pull and run from Docker Hub (after CI/CD deployment)
docker pull YOUR_DOCKERHUB_USERNAME/sales-analytics-app:latest
docker run -p 8501:8501 YOUR_DOCKERHUB_USERNAME/sales-analytics-app:latest
```

### Testing Commands
```bash
# Quick start: Run all tests
python -m pytest tests/ -v

# Install testing dependencies (if not in requirements.txt)
pip install pytest pytest-cov

# Run specific test categories
python -m pytest tests/test_analysis_functions.py -v    # Unit tests
python -m pytest tests/test_streamlit_integration.py -v # Integration tests  
python -m pytest tests/test_code_quality.py -v         # Code quality tests
python -m pytest tests/test_data_samples.py -v         # Data samples tests

# Run tests with coverage report
python -m pytest tests/ --cov=streamlit_app --cov-report=term-missing

# Run only fast tests (skip slow performance tests)
python -m pytest tests/ -v -m "not slow"

# Run performance tests
python -m pytest tests/ -v -m "performance"

# Run comprehensive test suite
python tests/run_tests.py

# Run with performance tests included
python tests/run_tests.py --performance
```

## Streamlit Web Application Features

The main `streamlit_app.py` provides:
- **File Upload**: Drag-and-drop Excel file upload interface
- **Sample Data**: Built-in button to load example dataset
- **Interactive Analysis**: Real-time data processing and visualization
- **Multiple Visualizations**: Time series plots, bar charts, correlation heatmaps
- **Statistical Reports**: Comprehensive analysis with downloadable reports
- **Responsive Design**: Clean, professional web interface in Russian

## Working with This Codebase

When working with this project:
- **Main Interface**: Use Streamlit app (`streamlit_app.py`) for interactive analysis
- **Language**: Interface and analysis in Russian with bilingual documentation
- **Data Format**: Excel files (.xlsx) with automatic format detection
- **Visualizations**: matplotlib and seaborn with Russian labels
- **Code Structure**: Streamlit follows modern Python practices, original code follows Jupyter format
- **Time Series**: Data spans from 2020 to 2025
- **Products**: Analysis focuses on three products: Продукт_1, Продукт_2, Продукт_3
- **Virtual Environment**: Always activate venv before running applications

## Testing Architecture

The project includes comprehensive testing with:

### Test Categories:
- **Unit Tests** (`test_analysis_functions.py`): Test individual functions with various data scenarios
- **Integration Tests** (`test_streamlit_integration.py`): Test full application workflow and component integration  
- **Code Quality Tests** (`test_code_quality.py`): Syntax validation, code structure, security checks
- **Data Sample Tests** (`test_data_samples.py`): Tests with realistic data scenarios and edge cases

### Test Features:
- **Fixtures**: Shared test data and configurations in `conftest.py`
- **Performance Testing**: Marked with `@pytest.mark.slow` and `@pytest.mark.performance`
- **Mocking**: Streamlit components mocked for unit testing
- **Coverage Analysis**: Code coverage reporting with pytest-cov
- **Automated Test Runner**: `tests/run_tests.py` for comprehensive testing

### Running Tests:
- Use `python tests/run_tests.py` for full automated test suite
- Individual test files can be run separately with pytest
- Performance tests are optional and marked separately

## Known Issues

1. **Legacy Script Path**: The original `docs/пример1.py` references Google Colab paths
   - **Not an issue**: Legacy script not used in production Streamlit app
   - **Streamlit app**: Handles file uploads and local paths automatically
2. **Character Encoding**: Windows console may have issues displaying Cyrillic characters
3. **Virtual Environment**: Script may fail if dependencies not installed in activated venv
4. **Docker Production**: Fully self-contained - only includes `streamlit_app.py` (sample data generated programmatically)

## CI/CD Pipeline

### GitHub Actions Workflow
The project includes automated CI/CD pipeline (`.github/workflows/ci-cd.yml`) that:

**On Pull Requests to `main`:**
- ✅ Runs all tests (unit, integration, quality)
- ✅ Checks code syntax with flake8
- ✅ Generates coverage reports
- ✅ Builds Docker image for validation

**On Push to `main` branch:**
- ✅ Runs complete test suite
- ✅ Builds multi-platform Docker image (amd64, arm64)
- ✅ Pushes to Docker Hub with tags (latest, branch, SHA)
- ✅ Scans for security vulnerabilities
- ✅ Deploys to production environment

### Required GitHub Secrets
Set these in your GitHub repository settings:
- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_TOKEN`: Your Docker Hub access token

### Git Workflow Strategy
**Recommended approach:**
1. **Development in `dev` branch**: All development work, feature branches
2. **Production in `main` branch**: Stable, tested code only
3. **Pull Requests**: From `dev` → `main` trigger testing
4. **Automatic Deployment**: Push to `main` triggers Docker Hub deployment

**Workflow:**
```bash
# Work in dev branch
git checkout dev
git add .
git commit -m "Add new features"
git push origin dev

# When ready for production:
git checkout main
git merge dev          # or create PR: dev → main
git push origin main   # Triggers CI/CD deployment
```

### Docker Hub Integration
After successful CI/CD:
- Image available as: `YOUR_USERNAME/sales-analytics-app:latest`
- Tagged with branch name and commit SHA
- Multi-platform support (Linux AMD64/ARM64)
- Automatic security scanning

## File Naming Convention

Files use Cyrillic names (`пример1`) reflecting the Russian language context of the project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mihnin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mihnin)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
