## regime-detection-strategy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository implements a Market Regime Detection & Adaptive Strategy Selection system using machine learning. The system detects market regimes through unsupervised learning (GMM, HMM, DTW clustering), predicts regime transitions with supervised ML, and dynamically selects trading strategies based on current market conditions.

## Architecture

### Core Modules

**src/data/**: Data acquisition, preprocessing, and feature engineering
- `data_loader.py`: Multi-source data fetching (Yahoo Finance, Alpha Vantage) with caching
- `data_preprocessor.py`: Data cleaning, outlier removal, validation
- `feature_engineer.py`: 50+ technical indicators across 5 categories (trend, volatility, momentum, volume, statistical)

**src/regime_detection/**: Machine learning models for regime identification
- `gmm_detector.py`: Gaussian Mixture Models with BIC/AIC optimization
- `hmm_detector.py`: Hidden Markov Models with Viterbi and forward-backward algorithms
- `dtw_clustering.py`: Dynamic Time Warping-based clustering
- `transition_predictor.py`: Random Forest/XGBoost for regime transition prediction

**src/strategies/**: Trading strategy framework
- `base_strategy.py`: Abstract base class for all strategies
- `strategy_selector.py`: Maps regimes to optimal strategies

**src/utils/**: Utilities for visualization and performance metrics
- `plotting.py`: Regime overlay plots, equity curves
- `metrics.py`: Sharpe, Sortino, Calmar ratios, max drawdown

### Key Design Patterns

1. **Modular Architecture**: Each component is independent and composable
2. **Strategy Pattern**: BaseStrategy allows easy addition of new strategies
3. **Factory Pattern**: StrategySelector maps regimes to strategies
4. **Caching**: DataLoader caches downloaded data in parquet format
5. **Standardization**: All models use StandardScaler for features

## Development Commands

### Environment Setup
```bash
# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
```

### Testing
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src --cov-report=html --cov-report=term

# Run specific test file
pytest tests/unit/test_data_loader.py

# Run integration tests
pytest tests/integration/
```

### Code Quality
```bash
# Format code
black src/ tests/

# Lint
pylint src/
flake8 src/ --max-line-length=100

# Type checking
mypy src/
```

## Key Implementation Details

### Feature Engineering Pipeline
The FeatureEngineer class creates features in specific groups. When adding new features:
1. Add to appropriate `_add_*_features()` method
2. Update `extract_regime_features()` if regime-relevant
3. Update `get_feature_names()` for proper categorization
4. Add unit tests in `tests/unit/test_feature_engineer.py`

### Regime Detection Workflow
Standard workflow for regime detection:
```python
# 1. Load and preprocess data
loader = DataLoader()
data = loader.load_data('SPY', start_date='2020-01-01')

preprocessor = DataPreprocessor()
clean_data = preprocessor.clean_data(data)

# 2. Engineer features
engineer = FeatureEngineer()
features = engineer.create_features(clean_data)
regime_features = engineer.extract_regime_features(features)

# 3. Detect regimes
detector = GMMDetector(n_regimes=3)
detector.fit(regime_features)
regimes = detector.predict(regime_features)

# 4. Analyze results
stats = detector.get_regime_statistics(regime_features, returns=data['close'].pct_change())
```

### Adding New Regime Detection Methods
1. Inherit from appropriate base (or create new abstract base)
2. Implement `fit()`, `predict()`, `predict_proba()` methods
3. Add to `src/regime_detection/__init__.py`
4. Create corresponding unit tests
5. Update API documentation

## Testing Strategy

- **Unit Tests**: Test individual functions/methods in isolation
- **Integration Tests**: Test complete workflows (data → features → regimes → strategies)
- **Fixtures**: Use pytest fixtures for sample data
- **Mocking**: Mock external API calls (yfinance) to avoid network dependencies
- **Coverage Goal**: >80% line coverage for all modules

## Git Workflow

Follow conventional commits:
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `test:` Tests
- `refactor:` Code refactoring

Make atomic commits with clear messages. Push regularly to maintain backup.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sakeeb91)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sakeeb91)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
