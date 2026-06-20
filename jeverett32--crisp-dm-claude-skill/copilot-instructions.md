## crisp-dm-claude-skill

> Builds a production-grade ML pipeline following a staged CRISP-DM framework. Uses a modular 'src/' + independent notebook architecture. Employs staged interviews to minimize hallucinations and gather comprehensive context per phase. Outputs individual self-contained phase notebooks AND a comprehensive, runnable Master notebook with an executive summary.


# CRISP-DM Staged Machine Learning Pipeline

This skill generates a complete, end-to-end ML pipeline grounded in the CRISP-DM framework (problem framing → data understanding → preparation → modeling → evaluation), and can optionally "operationalize" that pipeline into a repeatable training + inference workflow.

## Key Design Principles

1. **Staged Interviews**: The process is NOT a single initial interview. We pause after each phase to conduct a targeted interview for the next phase. This gives the agent richer context and less room to hallucinate.
2. **Modular Architecture**: All reusable logic (data loading, preprocessing, modeling, evaluation) lives in `src/`. Notebooks are thin interfaces that import from `src/`, ensuring consistency and independence.
3. **Independent Notebooks**: Each phase notebook AND the Master notebook must be self-contained and runnable independently. They check for/load intermediate artifacts or regenerate them if missing.
4. **Sign-off Gates**: Before proceeding to the next phase, the agent must present a "Phase Summary & Conclusion" for explicit user sign-off.

---

## Step 0: Initialize a Lab Tree (workspace for all outputs)

Before writing analysis/code artifacts, create a **lab workspace folder** in the repo so the project stays organized and repeatable.

### Lab tree rules
- Put **all generated code, notebooks, and reports under the lab tree** (do not scatter files at repo root).
- Do not modify existing application code outside the lab tree unless the user explicitly asks.
- Use a short, filesystem-safe project slug derived from the user's goal, e.g. `churn_prediction`, `late_delivery`, `house_prices`.

### Standard lab tree layout (create if missing)

```
lab/
  <project_slug>/
    README.md
    requirements.txt              # or pyproject.toml if project uses it
    data/
      raw/                        # immutable inputs (or links/notes)
      interim/                    # intermediate transforms
      processed/                  # modeling-ready tables
    notebooks/
      01_business_understanding.ipynb
      02_data_understanding.ipynb
      03_data_preparation.ipynb
      04_modeling.ipynb
      05_evaluation.ipynb
      master_crispdm_pipeline.ipynb   # Independent, stakeholder-ready
    src/
      __init__.py
      config.py                   # paths, constants, random seeds
      data_io.py                  # loading utilities
      features.py                 # feature engineering (shared by train + infer)
      metrics.py                  # metric helpers/baselines
      modeling.py                 # model definitions / tuning helpers
      evaluation.py               # evaluation plots/reports
    jobs/                         # operationalization (optional)
      etl_build_warehouse.py
      train_model.py
      run_inference.py
      utils_db.py                 # if DB source/sink is used
    artifacts/
      models/                     # saved model pipelines (.sav)
      runs/                       # per-run metadata/metrics
    reports/
      figures/
      tables/
      executive_summary.md
    logs/
```

If the user does not want operationalization, you can omit creating `jobs/` initially, but keep the lab tree structure so the work remains organized.

---

## Step 1: The Staged Interview Protocol

**CRITICAL**: Do NOT ask all questions up front. Follow this staged approach. The agent gathers context progressively, which leads to better-informed questions and less hallucination.

### Phase 0: Initial Intake (ask once, at the very beginning)

Extract what you can from the conversation context — only ask what's genuinely missing.

**Minimum viable set:**
1. **Goal & decision**: What decision will this model support, and who will use it?
2. **Data source & access**:
   - **Files**: path(s) (CSV/Excel/Parquet), or
   - **Database**: engine (SQLite/Postgres/etc), file/connection info, and table(s), or
   - **Multiple sources** that need joining/denormalizing?
3. **Target definition**:
   - Target column name (or how to construct the label)
   - If classification: what is the **positive class** (the "important" class)?
4. **Output choice**:
   - **Operationalization** (repeatable train + infer workflow): **Yes/No**
     - If Yes: where should predictions go? (file, database table, app integration point)

### The Phase Loop (Repeat for Phases 1-5)

For each CRISP-DM phase, follow this sequence:
1. **Interview**: Conduct the phase-specific deep-dive interview (see Staged Probing Schedule below).
2. **Implementation**: Generate/update the phase-specific notebook and relevant `src/` modules.
3. **Verification**: Present the phase results to the user.
4. **Sign-off**: Request explicit user sign-off on the phase conclusions before proceeding.

**Do not move to the next phase until the user has explicitly approved the current phase.**

### Staged Probing Schedule

| After This Phase | Conduct This Interview (for the NEXT phase) |
|---|---|
| **Initial Intake** | Proceed to Phase 1 (Business Understanding) |
| **Phase 1: Business** | **Stakeholders & approval chains**: Who are the stakeholders? Who needs to approve? **Automation scope**: What decisions will be automated vs. human-in-the-loop? **Business success**: What does "success" look like in business terms? (revenue, cost avoidance, etc.) **Failure history**: Are there known problem cases or examples of failures? |
| **Phase 2: Data** | **Data dictionary & experts**: Do you have a data dictionary? Domain expert contacts? **Quality issues**: Are there known data quality issues? **Temporal scope**: What's the temporal scope of the data? **Seasonality/drift**: Are there seasonal patterns or known drift periods? |
| **Phase 3: Prep** | **Outliers & errors**: Known outliers or erroneous values? **Feature hypotheses**: Feature engineering ideas or hypotheses? **Derived metrics**: Are there derived metrics already computed elsewhere? **Compliance/fairness**: Any compliance/fairness constraints on features? |
| **Phase 4: Modeling** | **Interpretability vs. performance**: Preference for interpretable vs. higher-performance models? **Constraints**: Inference latency, model size, deployment environment? **Prior attempts**: Any prior attempts that failed and why? |
| **Phase 5: Evaluation** | **Segment performance**: What segment performance matters most? **Model staleness**: What's the cost of model updates vs. stale models? **Monitoring**: How will model be monitored in production? |

### Decision rules (don't re-ask later)
- If **problem type** is unclear, infer from target dtype/values and confirm in a single sentence.
- If the user doesn't know a metric, choose a default based on error costs:
  - **Classification**: prefer F1 / recall / ROC AUC depending on "misses vs false alarms"
  - **Regression**: MAE or RMSE depending on sensitivity to large errors

### Anti-Hallucination Rule
**If you do not have explicit user confirmation for a key design choice (e.g., success metric, feature set, model type), you MUST ask before proceeding.** Document the user's explicit answer in the notebook's "Phase Evidence & Assumptions" section. Never invent constraints or assumptions without flagging them for user review.

---

## Problem Type Adaptation

**CRITICAL**: Before generating any phase, determine the problem type and apply the correct adaptations below. Do NOT use the default templates blindly — modify each phase based on the data type, problem type, and data characteristics.

### Problem Type Detection
- **Binary Classification**: Target has exactly 2 classes
- **Multi-class Classification**: Target has 3+ classes
- **Regression**: Target is continuous numeric
- **Time Series Forecasting**: Data has a temporal ordering and the target is a future value of a time-dependent variable
- **Time Series Classification/Regression**: Tabular data with a time component where rows are ordered chronologically but the task is not pure forecasting

### Adaptation Matrix

| Dimension | Non-Time-Series | Time Series |
|---|---|---|
| **Split Strategy** | `train_test_split` (random or stratified) | Chronological split (last N% as test) |
| **Cross-Validation** | `StratifiedKFold` / `KFold` | `TimeSeriesSplit` or rolling window |
| **Feature Engineering** | Domain features, interactions | Lag features, rolling stats, seasonal decompositions, date-time features |
| **EDA** | Correlations, distributions | Autocorrelation (ACF/PACF), trend decomposition, seasonality plots |
| **Leakage Prevention** | Standard column exclusion | No future data in features; respect temporal ordering |
| **Baseline** | Majority class / mean prediction | Naive forecast (last value), seasonal naive, or rolling mean |
| **Models** | LogisticRegression, RandomForest, XGBoost, etc. | ARIMA/SARIMAX, Prophet, LightGBM with lag features, TemporalFusionTransformer |

### Imbalanced Data Handling
When classification target class imbalance ratio exceeds 4:1:
- **Phase 1**: Document the imbalance and its business implications. Ask about cost asymmetry.
- **Phase 3**: Apply one or more of: `class_weight="balanced"` in models, SMOTE via `imblearn`, or threshold tuning.
- **Phase 4**: Use `roc_auc`, `f1`, or `average_precision` as scoring — never `accuracy` alone.
- **Phase 5**: Report per-class precision/recall, not just overall metrics. Include PR curve alongside ROC.

### Multi-class Classification Handling
- **Phase 1**: Identify all classes and their business meaning. Ask which misclassifications are most costly.
- **Phase 3**: Use `stratify=y` in split only if all classes have sufficient samples.
- **Phase 4**: Use `StratifiedKFold`. Models: `RandomForestClassifier`, `XGBClassifier`, `LogisticRegression` (supports multi-class natively).
- **Phase 5**: Use `classification_report` (shows per-class metrics). Include confusion matrix with normalized view.

---

## Canonical Phase Templates (self-contained)

Use these templates as the canonical structure. Adapt variable names, file paths, and model choices to the user's context.

### Phase 1: Business Understanding
**CRISP-DM Purpose:** define the business problem, objectives, and success criteria before touching data.

**Deliverables:**
- Problem statement and scope
- Feasibility framing: practical impact, data availability, analytical feasibility
- Success metric and baseline to beat
- Error cost analysis and metric justification
- Stakeholder impact assessment (revenue, cost avoidance, etc.)

**Notebook markdown header template:**

```markdown
## Phase 1: Business Understanding
**CRISP-DM Purpose:** Define the business problem, objectives, and success criteria before any data work begins.

### Problem Statement
- **Business Question:** ...
- **Target Variable:** `...`
- **Problem Type:** Classification / Regression
- **Positive Class (if classification):** ...

### Feasibility Assessment
| Criterion | Assessment |
|---|---|
| Practical Impact | ... |
| Data Availability | ... |
| Analytical Feasibility | ... |

### Success Criteria
- **Primary metric:** ...
- **Baseline to beat:** ...
- **Minimum acceptable performance:** ...

### Error Cost Analysis
- **False Positive cost:** ...
- **False Negative cost:** ...
- **Implication for metric choice:** ...

### Stakeholder Impact
- **Expected business value:** ...
- **Decisions to be automated:** ...
- **Human-in-the-loop points:** ...
```

### Phase 2: Data Understanding
**CRISP-DM Purpose:** become familiar with the data's structure, variables, quality, and relationships.

**Deliverables:**
- Data description report (shape, dtypes, sample rows)
- Univariate statistics table
- Missing value report
- Target distribution
- Data quality issues list (identified, not fixed yet)
- Relationship exploration (correlations/plots)
- Temporal scope and seasonality analysis

**Canonical code pattern (non-time-series):**

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings("ignore")

# Load
df = pd.read_csv("PATH_TO_DATA.csv")  # or database query result
print(f"Shape: {df.shape[0]:,} rows × {df.shape[1]} columns")
display(df.head()) if "display" in globals() else print(df.head())

# Univariate stats + missingness
desc = df.describe(include="all").T
desc["missing"] = df.isnull().sum()
desc["missing_pct"] = (df.isnull().mean() * 100).round(2)
desc["nunique"] = df.nunique()
print(desc[["count", "missing", "missing_pct", "nunique"]].head(30))

# Target distribution + baseline
TARGET = "YOUR_TARGET"
if df[TARGET].dtype == "O" or df[TARGET].nunique() <= 20:
    vc = df[TARGET].value_counts()
    print(vc)
    print(f"Baseline accuracy (majority class): {vc.max() / vc.sum():.1%}")
else:
    print(df[TARGET].describe())
```

**Canonical code pattern (time series EDA — add to or replace above when data is time-ordered):**

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
from statsmodels.tsa.seasonal import seasonal_decompose
import warnings
warnings.filterwarnings("ignore")

# Load and ensure datetime index
df = pd.read_csv("PATH_TO_DATA.csv", parse_dates=["DATE_COLUMN"])
df = df.sort_values("DATE_COLUMN").reset_index(drop=True)
df.set_index("DATE_COLUMN", inplace=True)

print(f"Shape: {df.shape[0]:,} rows × {df.shape[1]} columns")
print(f"Date range: {df.index.min()} to {df.index.max()}")
print(f"Frequency estimate: {pd.infer_freq(df.index)}")

# Target time series plot
TARGET = "YOUR_TARGET"
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
df[TARGET].plot(ax=axes[0, 0], title=f"{TARGET} over time")
df[TARGET].resample("M").mean().plot(ax=axes[0, 1], title="Monthly trend")
plot_acf(df[TARGET].dropna(), ax=axes[1, 0], title="Autocorrelation (ACF)")
plot_pacf(df[TARGET].dropna(), ax=axes[1, 1], title="Partial Autocorrelation (PACF)")
plt.tight_layout()
plt.show()

# Seasonal decomposition (adjust period for your data)
decomp = seasonal_decompose(df[TARGET].dropna(), model="additive", period=7)  # or 12, 52, etc.
decomp.plot()
plt.show()

# Rolling statistics
df["rolling_mean_7"] = df[TARGET].rolling(window=7).mean()
df["rolling_std_7"] = df[TARGET].rolling(window=7).std()
print(df[[TARGET, "rolling_mean_7", "rolling_std_7"]].tail(20))
```

### Phase 3: Data Preparation
**CRISP-DM Purpose:** transform raw data into modeling-ready form; fixes happen here.

**Deliverables:**
- Inclusion/exclusion report (dropped columns + reasons)
- Cleaning decisions (imputation/outliers)
- Feature engineering notes
- Train/test split (frozen test set)
- Leakage-safe preprocessing using `Pipeline` + `ColumnTransformer`
- Compliance/fairness constraint documentation

**Canonical rules:**
- Freeze test set once; do all tuning/CV on train only.
- No preprocessing fit on full dataset outside of a pipeline.
- **For time series**: split chronologically, never randomly. No future data in features.

**Canonical code pattern (non-time-series):**

```python
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer

SEED = 42
TARGET = "YOUR_TARGET"
EXCLUDE_COLS = []  # leakage IDs, timestamps, etc.

df_clean = df.drop(columns=[c for c in EXCLUDE_COLS if c in df.columns])
X = df_clean.drop(columns=[TARGET])
y = df_clean[TARGET]

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=SEED, stratify=y if y.nunique() <= 20 else None
)

numeric_cols = X_train.select_dtypes(include=np.number).columns.tolist()
categorical_cols = X_train.select_dtypes(exclude=np.number).columns.tolist()

numeric_pipe = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("scale", StandardScaler()),
])
categorical_pipe = Pipeline([
    ("impute", SimpleImputer(strategy="most_frequent")),
    ("onehot", OneHotEncoder(handle_unknown="ignore", sparse_output=False)),
])

preprocessor = ColumnTransformer(
    [("num", numeric_pipe, numeric_cols), ("cat", categorical_pipe, categorical_cols)],
    remainder="drop",
)
```

**Canonical code pattern (time series — chronological split + lag features):**

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer

SEED = 42
TARGET = "YOUR_TARGET"
DATE_COL = "YOUR_DATE_COLUMN"
EXCLUDE_COLS = [DATE_COL]  # date column excluded from features

# Sort chronologically
df = df.sort_values(DATE_COL).reset_index(drop=True)

# Create lag features BEFORE splitting
LAGS = [1, 2, 3, 7, 14, 28]  # adjust to your domain
for lag in LAGS:
    df[f"{TARGET}_lag_{lag}"] = df[TARGET].shift(lag)

# Rolling statistics
df[f"{TARGET}_roll_mean_7"] = df[TARGET].rolling(window=7).mean()
df[f"{TARGET}_roll_std_7"] = df[TARGET].rolling(window=7).std()

# Date-time features
df["day_of_week"] = df[DATE_COL].dt.dayofweek
df["month"] = df[DATE_COL].dt.month
df["quarter"] = df[DATE_COL].dt.quarter

# Drop rows with NaN from lagging
df = df.dropna().reset_index(drop=True)

# Chronological split (last 20% as test)
split_idx = int(len(df) * 0.8)
df_train = df.iloc[:split_idx]
df_test = df.iloc[split_idx:]

X_train = df_train.drop(columns=[TARGET] + EXCLUDE_COLS)
y_train = df_train[TARGET]
X_test = df_test.drop(columns=[TARGET] + EXCLUDE_COLS)
y_test = df_test[TARGET]

numeric_cols = X_train.select_dtypes(include=np.number).columns.tolist()
categorical_cols = X_train.select_dtypes(exclude=np.number).columns.tolist()

numeric_pipe = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("scale", StandardScaler()),
])
categorical_pipe = Pipeline([
    ("impute", SimpleImputer(strategy="most_frequent")),
    ("onehot", OneHotEncoder(handle_unknown="ignore", sparse_output=False)),
])

preprocessor = ColumnTransformer(
    [("num", numeric_pipe, numeric_cols), ("cat", categorical_pipe, categorical_cols)],
    remainder="drop",
)
```

### Phase 4: Modeling
**CRISP-DM Purpose:** train/compare candidate models using CV on training set only; tune without touching test.

**Deliverables:**
- Candidate techniques + assumptions
- CV design specification (and rationale)
- CV comparison table
- Selected model + tuned parameters
- Interpretability vs. performance trade-off documentation

**Canonical patterns (non-time-series):**
- Classification: `StratifiedKFold`, choose scoring aligned to Phase 1 costs.
- Regression: `KFold`, use `neg_root_mean_squared_error` or `r2` as appropriate.
- Imbalanced data: use `roc_auc`, `f1`, or `average_precision` scoring; apply `class_weight="balanced"` or SMOTE.
- Multi-class: use `StratifiedKFold`; models support multi-class natively.

```python
from sklearn.model_selection import StratifiedKFold, KFold, cross_val_score, GridSearchCV
from sklearn.pipeline import Pipeline

# Pick based on problem type
is_classification = (y_train.nunique() <= 20)
CV = StratifiedKFold(n_splits=5, shuffle=True, random_state=SEED) if is_classification else KFold(n_splits=5, shuffle=True, random_state=SEED)
SCORING = "roc_auc" if is_classification else "neg_root_mean_squared_error"

# Example candidates (swap as needed)
from sklearn.linear_model import LogisticRegression, Ridge
from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor

candidates = {}
if is_classification:
    candidates["LogReg"] = Pipeline([("prep", preprocessor), ("model", LogisticRegression(max_iter=2000, random_state=SEED))])
    candidates["RF"] = Pipeline([("prep", preprocessor), ("model", RandomForestClassifier(n_estimators=300, random_state=SEED, n_jobs=-1))])
else:
    candidates["Ridge"] = Pipeline([("prep", preprocessor), ("model", Ridge(alpha=1.0))])
    candidates["RF"] = Pipeline([("prep", preprocessor), ("model", RandomForestRegressor(n_estimators=300, random_state=SEED, n_jobs=-1))])

for name, pipe in candidates.items():
    scores = cross_val_score(pipe, X_train, y_train, cv=CV, scoring=SCORING, n_jobs=-1)
    print(name, float(scores.mean()), float(scores.std()))

# Optional tuning on best candidate
# search = GridSearchCV(best_pipe, param_grid, cv=CV, scoring=SCORING, n_jobs=-1)
# search.fit(X_train, y_train)
# final_model = search.best_estimator_
```

**Canonical patterns (time series):**
- Use `TimeSeriesSplit` for CV. Never shuffle.
- Include gradient boosting models (XGBoost, LightGBM) as primary candidates — they handle lag features well.
- For pure forecasting, also try statistical baselines (naive, seasonal naive).

```python
from sklearn.model_selection import TimeSeriesSplit, cross_val_score
from sklearn.pipeline import Pipeline

CV = TimeSeriesSplit(n_splits=5)
SCORING = "neg_root_mean_squared_error"  # or "r2" for regression, "roc_auc" for classification

# Time series candidates
try:
    import xgboost as xgb
    from sklearn.linear_model import LinearRegression
    candidates = {
        "LinearReg": Pipeline([("prep", preprocessor), ("model", LinearRegression())]),
        "XGBoost": Pipeline([("prep", preprocessor), ("model", xgb.XGBRegressor(n_estimators=300, random_state=SEED, n_jobs=-1))]),
    }
except ImportError:
    from sklearn.ensemble import RandomForestRegressor
    candidates = {
        "RF": Pipeline([("prep", preprocessor), ("model", RandomForestRegressor(n_estimators=300, random_state=SEED, n_jobs=-1))]),
    }

for name, pipe in candidates.items():
    scores = cross_val_score(pipe, X_train, y_train, cv=CV, scoring=SCORING, n_jobs=-1)
    print(name, float(scores.mean()), float(scores.std()))

# Naive baseline for comparison
from sklearn.metrics import mean_absolute_error
naive_pred = X_test[f"{TARGET}_lag_1"].values if f"{TARGET}_lag_1" in X_test.columns else np.full(len(y_test), float(y_train.iloc[-1]))
print(f"Naive baseline MAE: {mean_absolute_error(y_test, naive_pred):.4f}")
```

### Phase 5: Evaluation
**CRISP-DM Purpose:** evaluate against business objectives; compare to baseline; make go/no-go recommendation.

**Deliverables:**
- Final metrics vs baseline and success threshold
- Confusion matrix + interpretation (classification) OR residual analysis (regression)
- Feature importance / interpretability summary (as applicable)
- Operational readiness review + decision
- Executive summary (`reports/executive_summary.md`)
- Production monitoring plan

**Canonical code pattern (classification):**

```python
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix

final_model.fit(X_train, y_train)
y_pred = final_model.predict(X_test)

baseline = y_test.value_counts(normalize=True).max()
acc = accuracy_score(y_test, y_pred)
print(f"Baseline accuracy: {baseline:.4f}")
print(f"Model accuracy:    {acc:.4f}")
print(classification_report(y_test, y_pred))
print(confusion_matrix(y_test, y_pred))
```

**Canonical code pattern (regression):**

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

final_model.fit(X_train, y_train)
y_pred = final_model.predict(X_test)

baseline_pred = np.full(shape=len(y_test), fill_value=float(y_train.mean()))
baseline_rmse = mean_squared_error(y_test, baseline_pred, squared=False)
rmse = mean_squared_error(y_test, y_pred, squared=False)
mae = mean_absolute_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)
print(f"Baseline RMSE: {baseline_rmse:.4f} | Model RMSE: {rmse:.4f} | MAE: {mae:.4f} | R2: {r2:.4f}")
```

**Canonical code pattern (time series forecasting — add to or replace above):**

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, mean_absolute_percentage_error

final_model.fit(X_train, y_train)
y_pred = final_model.predict(X_test)

# Naive baseline (last known value)
naive_pred = np.full(len(y_test), float(y_train.iloc[-1]))

mape = mean_absolute_percentage_error(y_test, y_pred)
naive_mape = mean_absolute_percentage_error(y_test, naive_pred)
rmse = mean_squared_error(y_test, y_pred, squared=False)
naive_rmse = mean_squared_error(y_test, naive_pred, squared=False)
mae = mean_absolute_error(y_test, y_pred)

print(f"Naive RMSE: {naive_rmse:.4f} | Model RMSE: {rmse:.4f}")
print(f"Naive MAPE: {naive_mape:.4f} | Model MAPE: {mape:.4f}")
print(f"Model MAE:  {mae:.4f}")

# Residual plot over time
residuals = y_test - y_pred
fig, axes = plt.subplots(1, 2, figsize=(14, 5))
pd.Series(y_test.values, index=y_test.index if hasattr(y_test, 'index') else range(len(y_test))).plot(ax=axes[0], label="Actual")
pd.Series(y_pred, index=y_test.index if hasattr(y_test, 'index') else range(len(y_test))).plot(ax=axes[0], label="Predicted")
axes[0].legend()
axes[0].set_title("Actual vs Predicted")
pd.Series(residuals).plot(ax=axes[1])
axes[1].axhline(0, color="red", linestyle="--")
axes[1].set_title("Residuals over time")
plt.tight_layout()
plt.show()
```

---

## Critical Rules (The "Why" Behind Them)

**Never fit preprocessing on the full dataset before splitting.** Always put imputation, scaling, and encoding inside a `sklearn.Pipeline` with a `ColumnTransformer`. Fitting a scaler on the full dataset leaks validation statistics into training, producing optimistically biased performance estimates — a mistake that has burned real projects.

**Freeze the test set once, touch it once.** All cross-validation and hyperparameter tuning happens on `X_train` only. The test set is evaluated exactly once, at the very end, to report final performance. Using the test set for model selection is a form of overfitting.

**Always report a baseline.** For classification, compute the no-skill baseline (majority class rate). For regression, compute baseline RMSE using the mean prediction. For time series, compute naive forecast (last value) and seasonal naive baselines. A model that doesn't beat its baseline provides no real value — the textbook calls this out explicitly.

**Use the right CV strategy for your data type.** `StratifiedKFold` for classification, `KFold` for regression, `TimeSeriesSplit` for time-ordered data. For time series, never shuffle — respect temporal ordering. For group-structured data (multiple rows per entity), use `GroupKFold` to avoid leakage.

**For time series, never use future data in features.** Lag features, rolling windows, and date-time features must only reference past data. A feature using `shift(-1)` or a rolling window centered on the current row is leakage.

**Handle imbalanced data explicitly.** When class ratio exceeds 4:1, use `class_weight="balanced"`, SMOTE, or threshold tuning. Report per-class metrics, not just accuracy. Use PR curves alongside ROC for imbalanced problems.

**Document decisions, not just code.** Each phase should explain *why* a choice was made (e.g., "Used median imputation because this feature is right-skewed — mean would be pulled by outliers"). This is the difference between a notebook that teaches and one that just runs.

---

## "Done-ness" Gates (do not stop early)

Before considering the task complete, ensure the output includes:
- **Phase 1**: explicit success metric + baseline + minimum acceptable threshold (even if proposed)
- **Phase 2**: missingness report + target distribution + at least one relationship exploration; **for time series**: ACF/PACF plots, trend decomposition, seasonality analysis
- **Phase 3**: frozen test set + leakage-safe preprocessing in a pipeline; **for time series**: chronological split, lag features, no future data leakage
- **Phase 4**: CV comparison across at least 2 candidate models (or justified single-model choice); **for time series**: naive baseline comparison, `TimeSeriesSplit` CV
- **Phase 5**: final test evaluation + baseline comparison + go/no-go recommendation; **for imbalanced data**: per-class metrics + PR curve; **for time series**: residual plot over time + naive forecast comparison

If operationalizing:
- **Artifacts saved**: model pipeline file + metrics + metadata
- **Training/inference separation**: distinct code paths with shared feature logic
- **Prediction sink implemented**: file/db table/app integration location

---

## Operationalization Checklist (apply when operationalizing)

When generating the operational job layout, ensure:
- **Shared transformations**: feature engineering and preprocessing live in shared code; do not duplicate logic between training and inference.
- **Artifacts are complete**: saved object includes preprocessing + model (prefer saving the full `sklearn.Pipeline`).
- **Metadata exists**: training timestamp, data source snapshot description, feature list, label definition, code/config version (at minimum).
- **Metrics are logged**: include baseline comparison and chosen success metric.
- **Inference writes outputs**: predictions stored where the "app" can read them (often a DB table keyed by entity id).

---
> Source: [jeverett32/crisp-dm-claude-skill](https://github.com/jeverett32/crisp-dm-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
