## quantrl-lab

> This file provides guidance to agents (i.e., ADAL) when working with code in this repository.

# AGENTS.md

This file provides guidance to agents (i.e., ADAL) when working with code in this repository.

## Essential Commands

### Package Management

This project uses **uv** (not Poetry). The lock file is `uv.lock`.

```bash
uv sync                                    # Core deps only
uv sync --extra dev --extra notebooks      # With dev/notebook extras
uv sync --all-extras                       # All optional features
source .venv/bin/activate
```

Optional dependency groups: `dev`, `notebooks`, `ml` (torch/transformers/litellm), `tuning`, `viz`, `storage`, `full`.

### Testing

```bash
uv run pytest -m "not integration"         # Unit tests only (CI/CD - no API keys needed)
uv run pytest -m integration               # Integration tests (requires .env with API keys)
uv run pytest tests/data/test_indicators.py  # Specific test file
uv run pytest -k "test_portfolio"          # Tests matching pattern
uv run pytest --cov=quantrl_lab            # With coverage
```

### Code Quality (Pre-commit Hooks)

Pre-commit hooks run automatically on `git commit`. Run manually with:

```bash
pre-commit install                         # One-time setup
pre-commit run --all-files
pre-commit run --files path/to/file.py     # Before committing changes
```

Hook configuration (`.pre-commit-config.yaml`):
- **Black**: `--line-length=120 --skip-string-normalization`
- **isort**: `--profile black`
- **flake8**: `--max-line-length=120`
- **docformatter**: Google-style, `--wrap-summaries=72`

**CRITICAL: Always verify pre-commit hooks pass after making changes.**
Common failures: unused imports (flake8), line length violations, missing/malformed docstrings.

### Documentation

```bash
uv run mkdocs serve    # Local preview
uv run mkdocs build    # Build (always run before committing API changes)
```

When renaming modules or changing public APIs, update `docs/api-reference/` to match actual module paths and run `mkdocs build` to verify.

**Standalone guide tabs** (top-level nav, each a self-contained reference):
- `docs/DATA_SOURCES.md` — data loaders, capability matrix, usage per source
- `docs/data-processing.md` — `DataProcessor` and `DataPipeline` with all 7 pipeline steps
- `docs/environments.md` — action/observation/reward spaces, all reward strategies, full env example
- `docs/experiments.md` — `BacktestRunner`, `ExperimentJob`, `JobGenerator`, `AgentExplainer`, `OptunaRunner`

### Notebooks

```bash
# Install Jupyter kernel (one-time)
python -m ipykernel install --user --name quantrl-lab --display-name "QuantRL-Lab"

# Start Jupyter
jupyter notebook
# Then select "QuantRL-Lab" kernel
```

**Key notebooks:**
- `notebooks/backtesting_example.ipynb` - Main workflow demo
- `notebooks/feature_selection.ipynb` - Feature engineering
- `notebooks/hyperparameter_tuning.ipynb` - Optuna tuning

### Development Gotchas

- **Never commit `.env`** - contains API keys (Alpaca, Alpha Vantage, etc.)
- **Use `.env.example` as template** for required environment variables
- **Python 3.10+ required** (see pyproject.toml)
- **Module imports**: `quantrl_lab.*` (package installed in editable mode)

## Architecture

### Core Design Pattern: Strategy Injection

The environment accepts 3 pluggable strategy objects at instantiation — the central pattern of the entire codebase:

```python
env = SingleStockTradingEnv(
    data=df,
    config=config,
    action_strategy=action_strategy,          # How actions are processed
    reward_strategy=reward_strategy,          # How rewards are calculated
    observation_strategy=observation_strategy  # What the agent observes
)
```

This decouples environment logic from algorithmic choices. Change reward functions without touching environment code. Swap observation features without rewriting state logic.

**Step execution order** (inside `env.step(action)`):
1. Store `prev_portfolio_value`
2. `portfolio.process_open_orders()` — execute pending limit/stop orders
3. `action_strategy.handle_action(env, action)` — decode & execute new order
4. Advance `current_step`, check `terminated`/`truncated`
5. `reward_strategy.calculate_reward(env)` — compute reward
6. Clip reward to `reward_clip_range`
7. `reward_strategy.on_step_end(env)` — stateful hook
8. `observation_strategy.build_observation(env)` — compute state

### Data Flow

```
DataLoader (Alpaca/YFinance/AlphaVantage/FMP)
  → get_historical_ohlcv_data() → DataFrame with OHLCV
  → DataProcessor.data_processing_pipeline() → DataFrame with indicators
  → SingleStockTradingEnv → step() delegates to strategies
  → RL Agent (PPO/SAC/A2C via stable-baselines3)
```

### Key Architectural Patterns

**1. Protocol-Based Data Sources** (`src/quantrl_lab/data/interface.py`)
Data sources implement capability protocols for runtime feature detection:
- `HistoricalDataCapable`, `LiveDataCapable`, `NewsDataCapable`, `StreamingCapable`, `FundamentalDataCapable`, `MacroDataCapable`, `AnalystDataCapable`
- Check capabilities: `loader.supported_features()` → `{'historical': True, 'live': False, ...}`

**2. Indicator Registry** (`src/quantrl_lab/data/indicators/registry.py`)
Technical indicators are auto-registered via decorator:
```python
@IndicatorRegistry.register('RSI')
def rsi(df: pd.DataFrame, window=14) -> pd.DataFrame: ...

df = IndicatorRegistry.apply('RSI', df, window=14)
IndicatorRegistry.list_all()  # ['SMA', 'EMA', 'RSI', 'MACD', ...]
```

**3. Composite Reward Pattern** (`src/quantrl_lab/environments/stock/strategies/rewards/composite.py`)
Combines reward components with configurable weights:
```python
reward_strategy = CompositeReward(
    strategies=[PortfolioValueChangeReward(), DifferentialSortinoReward(), InvalidActionPenalty()],
    weights=[0.5, 0.3, 0.2],
    normalize_weights=True,  # default; auto-normalises weights to sum to 1
    auto_scale=False,        # if True, z-scores each component before weighting
)
```

**4. Data Pipeline** (`src/quantrl_lab/data/processing/pipeline.py`)
Builder pattern for composable data transformations via `DataPipeline`.

### Source Structure

```
src/quantrl_lab/
├── environments/
│   ├── core/              # Base classes: interfaces.py, config.py, portfolio.py, types.py
│   └── stock/
│       ├── single.py      # SingleStockTradingEnv (main environment)
│       ├── multi.py       # Multi-stock environment
│       ├── components/    # Portfolio and config components
│       └── strategies/
│           ├── actions/   # standard.py, time_in_force.py
│           ├── observations/  # feature_aware.py, etc.
│           └── rewards/   # portfolio_value, sharpe, sortino, drawdown, turnover,
│                          # expiration, invalid_action, boredom, execution_bonus, composite
├── data/
│   ├── sources/           # alpaca_loader, yfinance_loader, alpha_vantage_loader, fmp_loader
│   ├── indicators/        # registry.py, technical.py
│   ├── processing/        # pipeline.py, processor.py
│   │   └── steps/         # cleaning/, features/, alternative/
│   ├── utils/             # date_parsing, symbol_handling, dataframe_normalization,
│   │                      # response_validation, request_utils, async_request_utils
│   ├── partitioning/      # ratio.py, date_range.py
│   └── interface.py       # Protocol definitions
├── experiments/
│   ├── backtesting/       # runner.py (BacktestRunner), training.py, evaluation.py,
│   │                      # builder.py, core.py, metrics.py, explainer.py, analysis.py
│   │   └── config/        # environment_config.py
│   └── tuning/            # optuna_runner.py
├── alpha_research/        # registry.py, models.py, analysis.py, metrics.py, alpha_strategies.py
├── screening/             # llm_hedge_screener.py, response_schemas.py, data_models.py, prompt.py
├── deployment/trading/    # alpaca_trader.py (live trading)
└── utils/                 # math.py

tests/                     # Pytest test suite (mirrors src/ structure)
├── conftest.py
├── data/
│   ├── sources/           # test_data_sources.py, test_data_sources_integration.py, test_async_loaders.py
│   ├── processing/steps/  # test_analyst.py, etc.
│   ├── utils/             # test_async_request_utils.py
│   └── test_indicators.py
├── environments/stock/
│   ├── strategies/rewards/  # test_boredom.py
│   ├── test_env.py, test_portfolio.py, test_action.py, test_reward.py
│   └── test_time_in_force_action.py
├── experiments/
│   ├── backtesting/       # test_builder.py, test_metrics.py, test_runner.py
│   └── tuning/            # test_optuna_runner.py
└── alpha_research/        # test_alpha_research.py

examples/end_to_end/       # End-to-end training scripts
├── shared/data_utils.py   # init_data_sources(), select_alpha_indicators(), process_symbol()
├── train_single_symbol.py, train_single_symbol_sac.py, train_single_symbol_a2c.py
├── train_multi_symbol.py, train_multi_symbol_sac.py, train_multi_symbol_a2c.py
└── tune_single_symbol.py

notebooks/                 # Usage example notebooks
```

## Common Workflows

### BacktestRunner Workflow

```python
# Preferred: fluent builder
from quantrl_lab.experiments.backtesting.builder import BacktestEnvironmentBuilder
env_config = (
    BacktestEnvironmentBuilder()
    .with_data(train_data=train_df, test_data=test_df)
    .with_strategies(action=..., reward=..., observation=...)
    .with_env_params(initial_balance=100_000, window_size=20)
    .build()
)

# Single job
job = ExperimentJob(algorithm_class=PPO, env_config=env_config, total_timesteps=50000)
runner = BacktestRunner(verbose=True)
result = runner.run_job(job)
BacktestRunner.inspect_result(result)

# Batch / grid
jobs = JobGenerator.generate_grid(algorithms=[PPO, SAC], env_configs={...}, total_timesteps=50000)
results = runner.run_batch(jobs)
BacktestRunner.inspect_batch(results)
```

`create_env_config_factory` still exists but is **DEPRECATED** — use `BacktestEnvironmentBuilder` instead.

### Adding New Components

**New technical indicator**: Add to `src/quantrl_lab/data/indicators/technical.py` with `@IndicatorRegistry.register('NAME')` decorator. Signature: `def name(df: pd.DataFrame, **kwargs) -> pd.DataFrame`.

**New reward strategy**: Inherit `BaseRewardStrategy` from `src/quantrl_lab/environments/core/interfaces.py`. Implement `calculate_reward(self, env: TradingEnvProtocol) -> float`. Optionally implement `on_step_end(env)` for state updates.

**New action strategy**: Inherit `BaseActionStrategy`. Implement `define_action_space()` and `handle_action(env_self, action)`.

**New observation strategy**: Inherit `BaseObservationStrategy`. Implement `define_observation_space(env)`, `build_observation(env)`, and `get_feature_names(env)`.

### Running Experiments

**Typical flow:**
1. Load data with DataLoader
2. Process features with `DataProcessor.data_processing_pipeline()`
3. Define strategies (action/observation/reward)
4. Build env config via `BacktestEnvironmentBuilder`
5. Create `ExperimentJob` and run via `BacktestRunner`
6. Analyze results with `inspect_result()` / `inspect_batch()`

**See:** `examples/end_to_end/` for complete training scripts, `notebooks/backtesting_example.ipynb` for notebook workflow.

### Reward Shaping Guidance

- Use `PortfolioValueChangeReward` as the primary signal with a minimal `TurnoverPenaltyReward`.
- Heavy penalties (Sortino, drawdown) cause "do nothing" convergence in short training runs.
- Available reward strategies: portfolio_value, sharpe, sortino, drawdown, turnover, expiration, invalid_action, boredom, execution_bonus, composite.

## Code Conventions

- **Line length**: 120 characters; single quotes (skip string normalization)
- **Docstrings**: Google-style, required on all public APIs
- **Type hints**: Required everywhere; use `Protocol` for interfaces
- **Python**: 3.10+ required

Google-style docstring format:
```python
def fn(param1: str, param2: int = 10) -> pd.DataFrame:
    """
    Brief description.

    Args:
        param1 (str): Description.
        param2 (int, optional): Description. Defaults to 10.

    Returns:
        pd.DataFrame: Description.

    Raises:
        ValueError: When something is invalid.
    """
```

## Non-Obvious Behaviors

- **`current_step` advances before reward**: reward sees the price *after* the action, not before
- **`action_type` set by `handle_action()`**: reward strategies read `env.action_type` and `env.decoded_action_info` which are set just before `calculate_reward()` is called
- **Portfolio resets on `reset()`** but running stats in stateful reward strategies (Sharpe, Sortino) do **not** reset between episodes by default — call `reward_strategy.reset()` manually if needed
- **Price column auto-detected**: searches 'close', 'Close', 'adj_close', or 4th column; override with `price_column=` arg
- **`window_size` is padding only**: observations at the start of an episode are padded by repeating the first row, not by slicing earlier data
- **`CompositeReward` weights not auto-normalised** unless `normalize_weights=True` (default); `auto_scale` running stats persist across episodes
- **`n_envs > 1`** only affects training (via `make_vec_env`); evaluation always uses a single env
- **Limit/stop orders with `OrderTIF.TTL`** expire after `order_expiration_steps` (default 5); GTC orders persist indefinitely

### Data Source Capabilities

- **Alpaca**: Historical, Live, Streaming, News (requires API keys)
- **YFinance**: Historical, Fundamentals (free, no key needed)
- **AlphaVantage**: Historical, Fundamentals, Macroeconomic, News (requires API key)
- **FMP**: Historical (daily + intraday), Analyst data (grades/ratings) (requires API key)

**Alpha Vantage free tier**: 25 req/day, 1 req/sec; `outputsize=full` and intraday require premium; loader auto-handles rate limiting.

**FMP**: single symbol per request; intraday timeframes: 5min, 15min, 30min, 1hour, 4hour.

## Environment Variables

Copy `.env.example` to `.env`:

```bash
ALPACA_API_KEY=...          # Alpaca (Historical, Live, Streaming, News)
ALPACA_SECRET_KEY=...
ALPACA_BASE_URL=https://paper-api.alpaca.markets
ALPHA_VANTAGE_API_KEY=...   # Fundamentals, Macroeconomic, News
FMP_API_KEY=...             # Intraday data, Analyst grades/ratings
OPENAI_API_KEY=...          # Optional: LLM hedge screener
```

## Key Files to Check First

**When modifying:**
- Action spaces → `src/quantrl_lab/environments/stock/strategies/actions/`
- Reward logic → `src/quantrl_lab/environments/stock/strategies/rewards/`
- Data loading → `src/quantrl_lab/data/sources/`
- Feature engineering → `src/quantrl_lab/data/processing/processor.py`
- Backtesting flow → `src/quantrl_lab/experiments/backtesting/runner.py`
- Alpha research → `src/quantrl_lab/alpha_research/`

**When debugging:**
- Check `StockPortfolio` state in `src/quantrl_lab/environments/stock/components/`
- Verify data array shape and `current_step` index
- Confirm strategy injection in environment `__init__`
- Review `BacktestRunner.inspect_result()` output

## External Dependencies

**Core ML stack:**
- `stable-baselines3` - PPO, SAC, A2C algorithms
- `gymnasium` - RL environment interface
- `optuna` - Hyperparameter tuning

**Data & analysis:**
- `pandas`, `numpy` - Data manipulation
- `yfinance`, `alpaca-py` - Market data
- `seaborn`, `matplotlib` - Visualization

**See `pyproject.toml` for complete list.**

## Future Improvements

### Long-term Improvements (2-3 days each)
1. **DataValidator**: Build a dedicated `DataValidator` class for systematic quality checks (nulls, price relationships, duplicates).
2. **FillNA Strategy**: Implement `FillStrategy` protocol to replace hard-coded fillna logic in `SentimentFeatureGenerator`.
3. **Caching**: Implement `CachedDataSource` wrapper to reduce API calls.

### Ongoing Improvements
- **Protocol Consistency**: Standardize on pure Protocol-based design (structural typing) vs ABCs.
- **Testing**: Add pytest fixtures for common data shapes and improve dependency injection.

---
> Source: [whanyu1212/QuantRL-Lab](https://github.com/whanyu1212/QuantRL-Lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
