## titanic-survival-demo

> The entire process is a clear **Sequential Pipeline**, orchestrated by the `project-orchestrator`:

### Interaction Relationships and Call Flow

The entire process is a clear **Sequential Pipeline**, orchestrated by the `project-orchestrator`:

1.  **Start:**
    * **User** ==>  `project-orchestrator`: Submit raw data and analysis requirements.

2.  **Phase 1: Data Preparation**
    * `project-orchestrator`  ==>  `data-engineer`: "Please process this raw data."
    * `data-engineer`  ==>  `project-orchestrator`: Return clean datasets and processing logs.

3.  **Phase 2: Exploration & Insights**
    * `project-orchestrator`  ==>  `data-analyst-eda`: "Please perform exploratory analysis on this clean data."
    * `data-analyst-eda`  ==>  `project-orchestrator`: Return EDA reports, charts, and preliminary insights.

4.  **Phase 3: Modeling & Prediction**
    * `project-orchestrator`  ==>  `ml-model-scientist`: "Based on this data and preliminary insights, please build predictive models, identify key parameters and optimal ranges."
    * `ml-model-scientist`  ==>  `project-orchestrator`: Return model results, key parameter lists, and optimal range analysis.

5.  **Phase 4: Integration & Delivery**
    * `project-orchestrator`  ==>  `technical-writer-qa`: "Please integrate all outputs into final reports and Notebooks, and perform quality assurance."
    * `technical-writer-qa`  ==>  `project-orchestrator`: Return final `.md` reports and `.ipynb` files.

6.  **End:**
    * `project-orchestrator`  ==>  **User**: Deliver final deliverables.

### Workspace Environment Data Architecture Blueprint
#### Core Design Principles

1.  **Project Isolation:** Each new analysis request should be an independent project folder to avoid cross-contamination.
2.  **Separation of Concerns:** Raw data, processed data, code, reports, models, etc., should be stored separately by category.
3.  **Immutability of Raw Data:** Once content is placed in the raw data folder `raw`, it should never be modified by any program, ensuring traceability of analysis.
4.  **Process-Oriented:** Folder numbering (00, 01, 02...) reflects the main data processing workflow.

-----

#### Directory Structure Blueprint

This is the standard structure that the `Project Lead` Agent should initialize when receiving a new analysis project (e.g., `project_sales_prediction`).

```plaintext
📁 project_sales_prediction/
│
├── 📄 README.md                # Project documentation, maintained by Project Lead
│
├── 📁 00_data/                  # All data storage area
│   ├── 📁 raw/                  # Raw data (immutable)
│   │   └── 📄 sales_data_2025.csv
│   ├── 📁 processed/            # Data cleaned and processed by Data Engineer
│   │   └── 📄 cleaned_sales_data.parquet
│   └── 📁 intermediate/         # (Optional) Temporary data files generated during intermediate processes
│
├── 📁 01_notebooks/             # Jupyter Notebook workspace
│   ├── 📁 exploratory/          # Data Analyst's exploratory analysis drafts
│   │   └── 📄 eda_sales_analysis.ipynb
│   ├── 📁 modeling/             # Data Scientist's model development and experiments
│   │   └── 📄 sales_prediction_model_dev.ipynb
│   └── 📁 final/                # Final executable Notebook organized by Technical Writer
│       └── 📄 executable_sales_analysis.ipynb
│
├── 📁 02_src/                   # Reusable Python source code storage
│   ├── 📄 __init__.py
│   ├── 📄 data_processing.py    # Data Engineer's data cleaning functions
│   ├── 📄 visualization.py      # Data Analyst's visualization helper functions
│   └── 📄 modeling.py           # Data Scientist's model training and evaluation functions
│
├── 📁 03_reports/               # Final output reports and charts
│   ├── 📁 figures/              # Charts used in reports (png, jpg, svg)
│   │   ├── 📄 feature_importance.png
│   │   └── 📄 sales_trends_by_month.png
│   └── 📄 final_report.md       # Final Markdown report produced by Technical Writer
│
└── 📁 04_models/                # Model files saved after training completion
    └── 📄 sales_predictor_v1.pkl
```

-----

### Python Execution Environment

- Python 3.12 via pyenv (`python3`)
- Jupyter available via pyenv (`jupyter`)
- Core libraries: pandas, numpy, scikit-learn, matplotlib, seaborn

Run a script: `python3 <script.py>`
Launch Jupyter: `jupyter notebook`

### Important Notes
- project-orchestrator is only responsible for planning, not for generating and executing code, only responsible for planning and then assigning tasks to other subagents
- Do not use plt.show(), but directly save charts as files, such as using plt.savefig()
- final_report.md can include images from 03_reports/figures with explanations

## 🎨 Core Features

### ✨ Intelligent Division of Labor
- Each Agent specializes in specific domains, ensuring professional quality
- Automated task handover and result validation

### 📈 Standardized Outputs
- Unified code style and document formats
- Reproducible analysis workflows and results

### 🔄 Flexible Expansion
- Modular design, easy to add new Agent roles
- Support for different types of data science projects

### 🛡️ Quality Assurance
- Built-in code checking and testing mechanisms
- Technical document review and quality control

## 📚 Design Principles

### 🏗️ Architecture Principles
1. **Project Independence**: Each analysis project is completely independent, avoiding cross-contamination
2. **Separation of Concerns**: Data, code, and reports are stored separately by category
3. **Raw Data Immutability**: Ensure traceability and reproducibility of analysis
4. **Process-Oriented**: Directory structure reflects the main data processing workflow

### 💻 Technical Standards
- **Python Environment**: Use Anaconda to manage package dependencies
- **Visualization**: Charts directly saved as files
- **Document Format**: Markdown reports paired with high-quality PNG charts
- **Version Control**: Git manages code, model files stored independently

---
> Source: [chenkonturek/titanic-survival-demo](https://github.com/chenkonturek/titanic-survival-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
