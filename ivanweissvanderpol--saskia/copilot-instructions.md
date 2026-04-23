## saskia

> AI-specific coding standards and patterns for ML implementations on the Keyera Data Platform


# AI Code Standards for Keyera Data Platform

## Core Principles

### Data Platform Integration
- All AI code must integrate seamlessly with the medallion architecture (Bronze-Silver-Gold)
- Use existing data quality frameworks and extend them for ML-specific validations
- Follow established solution architecture patterns with AI/ML-specific adaptations
- Leverage Unity Catalog for ML asset management and governance

### AI-Specific Code Organization
- Organize AI code following the established solution structure:
  ```
  solutions/ai_solution_name/
  ├── 01_bronze_data_extraction.py
  ├── 02_silver_feature_engineering.py
  ├── 03_gold_model_training.py
  ├── 04_model_inference.py
  ├── solution_code/
  │   ├── feature_engineering.py
  │   ├── model_training.py
  │   ├── model_evaluation.py
  │   ├── model_deployment.py
  │   └── ai_utils.py
  ├── solution_resources/
  │   ├── model_configs/
  │   ├── feature_configs/
  │   └── deployment_configs/
  └── solution_tests/
      ├── test_feature_engineering.py
      ├── test_model_training.py
      └── test_model_evaluation.py
  ```

## AI Code Standards

### Model Development Standards
- Use type hints for all AI/ML functions and classes
- Implement proper error handling for ML-specific exceptions
- Follow functional programming principles for feature engineering
- Use dataclasses for ML configuration objects
- Implement comprehensive logging for model training and inference

### Feature Engineering Standards
```python
from typing import Dict, List, Optional, Union
from dataclasses import dataclass
from pyspark.sql import DataFrame
from pyspark.sql.functions import col

@dataclass
class FeatureConfig:
    """Configuration for feature engineering pipeline."""
    feature_name: str
    source_columns: List[str]
    transformation_type: str
    parameters: Dict[str, Union[str, int, float]]
    validation_rules: Dict[str, float]

class BaseFeatureEngineer:
    """Base class for feature engineering implementations."""
    
    def __init__(self, config: FeatureConfig):
        self.config = config
        self.logger = self._setup_logger()
    
    def transform(self, df: DataFrame) -> DataFrame:
        """Transform input DataFrame with feature engineering."""
        # Validate input data quality
        self._validate_input_data(df)
        
        # Apply transformations
        transformed_df = self._apply_transformations(df)
        
        # Validate output features
        self._validate_output_features(transformed_df)
        
        return transformed_df
    
    def _validate_input_data(self, df: DataFrame) -> None:
        """Validate input data quality for feature engineering."""
        # Use existing data quality framework
        pass
    
    def _apply_transformations(self, df: DataFrame) -> DataFrame:
        """Apply feature transformations."""
        raise NotImplementedError("Subclasses must implement transformation logic")
    
    def _validate_output_features(self, df: DataFrame) -> None:
        """Validate output feature quality."""
        pass
```

### Model Training Standards
```python
from typing import Dict, Any, Optional, Tuple
import mlflow
from mlflow.models import ModelSignature

class BaseModelTrainer:
    """Base class for model training implementations."""
    
    def __init__(self, model_config: Dict[str, Any]):
        self.config = model_config
        self.logger = self._setup_logger()
        self.mlflow_client = mlflow.tracking.MlflowClient()
    
    def train(self, train_df: DataFrame, val_df: Optional[DataFrame] = None) -> Dict[str, Any]:
        """Train model with comprehensive tracking and validation."""
        with mlflow.start_run() as run:
            # Log model configuration
            mlflow.log_params(self.config)
            
            # Prepare data
            X_train, y_train = self._prepare_training_data(train_df)
            
            # Train model
            model = self._train_model(X_train, y_train)
            
            # Evaluate model
            metrics = self._evaluate_model(model, val_df)
            mlflow.log_metrics(metrics)
            
            # Log model artifacts
            self._log_model_artifacts(model, run.info.run_id)
            
            return {
                'model': model,
                'metrics': metrics,
                'run_id': run.info.run_id
            }
    
    def _prepare_training_data(self, df: DataFrame) -> Tuple[Any, Any]:
        """Prepare training data from DataFrame."""
        raise NotImplementedError("Subclasses must implement data preparation")
    
    def _train_model(self, X: Any, y: Any) -> Any:
        """Train the actual model."""
        raise NotImplementedError("Subclasses must implement model training")
    
    def _evaluate_model(self, model: Any, val_df: Optional[DataFrame]) -> Dict[str, float]:
        """Evaluate model performance."""
        raise NotImplementedError("Subclasses must implement model evaluation")
    
    def _log_model_artifacts(self, model: Any, run_id: str) -> None:
        """Log model and related artifacts to MLflow."""
        # Register model in Unity Catalog
        # Save model artifacts
        # Log feature importance if applicable
        pass
```

### Model Inference Standards
```python
class BaseModelPredictor:
    """Base class for model inference implementations."""
    
    def __init__(self, model_uri: str, feature_config: FeatureConfig):
        self.model = mlflow.pyfunc.load_model(model_uri)
        self.feature_config = feature_config
        self.logger = self._setup_logger()
    
    def predict(self, input_df: DataFrame) -> DataFrame:
        """Generate predictions with comprehensive validation."""
        # Validate input data
        self._validate_input_schema(input_df)
        
        # Apply feature engineering
        features_df = self._prepare_features(input_df)
        
        # Generate predictions
        predictions_df = self._generate_predictions(features_df)
        
        # Validate predictions
        self._validate_predictions(predictions_df)
        
        return predictions_df
    
    def _validate_input_schema(self, df: DataFrame) -> None:
        """Validate input data schema."""
        # Use existing schema validation patterns
        pass
    
    def _prepare_features(self, df: DataFrame) -> DataFrame:
        """Prepare features for inference."""
        # Apply same feature engineering as training
        pass
    
    def _generate_predictions(self, df: DataFrame) -> DataFrame:
        """Generate model predictions."""
        # Convert to appropriate format for model
        # Apply model
        # Convert back to DataFrame
        pass
    
    def _validate_predictions(self, df: DataFrame) -> None:
        """Validate prediction outputs."""
        # Check for null predictions
        # Validate prediction ranges
        # Check for data drift
        pass
```

## Data Quality for AI

### Feature Quality Validation
- Implement statistical validation for feature distributions
- Check for feature drift between training and inference
- Validate feature correlation matrices
- Monitor for data leakage in feature engineering

### Model Quality Standards
```python
class ModelQualityValidator:
    """Validate model quality and performance."""
    
    def validate_model_performance(self, model_metrics: Dict[str, float]) -> bool:
        """Validate model meets performance thresholds."""
        thresholds = {
            'accuracy': 0.85,
            'precision': 0.80,
            'recall': 0.80,
            'f1_score': 0.80
        }
        
        for metric, threshold in thresholds.items():
            if metric in model_metrics:
                if model_metrics[metric] < threshold:
                    raise ValueError(f"Model {metric} {model_metrics[metric]} below threshold {threshold}")
        
        return True
    
    def validate_prediction_quality(self, predictions_df: DataFrame) -> bool:
        """Validate quality of model predictions."""
        # Check for null predictions
        null_count = predictions_df.filter(col("prediction").isNull()).count()
        if null_count > 0:
            raise ValueError(f"Found {null_count} null predictions")
        
        # Check prediction distribution
        prediction_stats = predictions_df.select("prediction").describe().collect()
        
        return True
```

## Error Handling and Logging

### AI-Specific Exception Handling
```python
class AIProcessingError(Exception):
    """Base exception for AI processing errors."""
    pass

class FeatureEngineeringError(AIProcessingError):
    """Exception for feature engineering errors."""
    pass

class ModelTrainingError(AIProcessingError):
    """Exception for model training errors."""
    pass

class ModelInferenceError(AIProcessingError):
    """Exception for model inference errors."""
    pass

class DataDriftError(AIProcessingError):
    """Exception for data drift detection."""
    pass
```

### Comprehensive AI Logging
```python
import logging
from typing import Any, Dict

class AILogger:
    """Specialized logger for AI operations."""
    
    def __init__(self, name: str):
        self.logger = logging.getLogger(f"ai.{name}")
        self._setup_handlers()
    
    def log_feature_engineering_start(self, feature_name: str, input_shape: tuple):
        """Log start of feature engineering process."""
        self.logger.info(f"Starting feature engineering for {feature_name}, input shape: {input_shape}")
    
    def log_model_training_start(self, model_type: str, parameters: Dict[str, Any]):
        """Log start of model training."""
        self.logger.info(f"Starting {model_type} training with parameters: {parameters}")
    
    def log_model_performance(self, metrics: Dict[str, float]):
        """Log model performance metrics."""
        self.logger.info(f"Model performance metrics: {metrics}")
    
    def log_prediction_batch(self, batch_size: int, prediction_stats: Dict[str, float]):
        """Log prediction batch processing."""
        self.logger.info(f"Processed prediction batch: size={batch_size}, stats={prediction_stats}")
```

## Configuration Management

### AI Configuration Standards
- Use YAML configuration files for all AI parameters
- Implement configuration validation with Pydantic models
- Version configuration files alongside model versions
- Store configurations in Unity Catalog for governance

```python
from pydantic import BaseModel, validator
from typing import List, Dict, Optional

class AIConfig(BaseModel):
    """Base configuration for AI components."""
    solution_name: str
    version: str
    environment: str
    
    @validator('environment')
    def validate_environment(cls, v):
        if v not in ['dev', 'staging', 'prod']:
            raise ValueError('Environment must be dev, staging, or prod')
        return v

class FeatureEngineeringConfig(AIConfig):
    """Configuration for feature engineering."""
    features: List[Dict[str, Any]]
    validation_rules: Dict[str, float]
    output_schema: Dict[str, str]

class ModelTrainingConfig(AIConfig):
    """Configuration for model training."""
    model_type: str
    hyperparameters: Dict[str, Any]
    training_data_path: str
    validation_split: float
    performance_thresholds: Dict[str, float]

class ModelDeploymentConfig(AIConfig):
    """Configuration for model deployment."""
    model_name: str
    model_version: str
    deployment_target: str
    serving_config: Dict[str, Any]
```

## Testing Standards

### AI-Specific Testing Requirements
- Unit tests for all feature engineering functions
- Integration tests for end-to-end ML pipelines
- Model performance tests with known datasets
- Data drift detection tests
- Load testing for model inference endpoints

### Example Test Structure
```python
import pytest
from pyspark.sql import SparkSession

class TestFeatureEngineering:
    """Test suite for feature engineering components."""
    
    def setup_method(self):
        """Setup test environment."""
        self.spark = SparkSession.builder.appName("test").getOrCreate()
        self.test_data = self._create_test_data()
    
    def test_feature_transformation(self):
        """Test feature transformation logic."""
        # Test with known input/output
        # Validate feature quality
        # Check for edge cases
        pass
    
    def test_feature_validation(self):
        """Test feature validation rules."""
        # Test validation rules
        # Test error conditions
        # Test boundary cases
        pass

class TestModelTraining:
    """Test suite for model training components."""
    
    def test_model_training_pipeline(self):
        """Test complete model training pipeline."""
        # Test with synthetic data
        # Validate model convergence
        # Check performance metrics
        pass
    
    def test_model_reproducibility(self):
        """Test model training reproducibility."""
        # Train same model twice
        # Compare results
        # Validate consistency
        pass
```

## Performance Optimization

### Spark Optimization for AI
- Use appropriate partitioning strategies for ML datasets
- Optimize DataFrame operations for feature engineering
- Implement efficient data sampling strategies
- Use broadcast variables for lookup tables in feature engineering

### Memory Management
- Monitor memory usage during model training
- Implement data checkpointing for large ML pipelines
- Use appropriate caching strategies for frequently accessed features
- Clean up intermediate DataFrames to prevent memory leaks

## Documentation Requirements

### AI Component Documentation
- Document all feature engineering logic and business rationale
- Include model architecture diagrams and decision rationale
- Document data sources and feature dependencies
- Maintain model performance benchmarks and evaluation criteria

### Code Documentation Standards
```python
class FinancialRiskModel:
    """
    Financial risk assessment model for credit analysis.
    
    This model predicts credit default probability using:
    - Customer financial metrics from Entero ERP
    - Historical payment patterns from accounts receivable
    - Market indicators from Bank of Canada data
    
    Model Architecture:
        - Feature engineering: 45 features across 3 categories
        - Algorithm: Gradient Boosting with XGBoost
        - Training data: 2+ years of historical data
        - Validation: Time-series split with 6-month holdout
    
    Performance Metrics:
        - AUC-ROC: 0.87 (target: >0.85)
        - Precision@10%: 0.75 (target: >0.70)
        - Recall: 0.82 (target: >0.80)
    
    Business Impact:
        - Reduces manual credit review time by 60%
        - Improves credit decision accuracy by 25%
        - Enables proactive risk management
    """
    
    def predict_default_probability(self, customer_data: DataFrame) -> DataFrame:
        """
        Predict credit default probability for customers.
        
        Args:
            customer_data: DataFrame with customer financial data
                Required columns:
                - customer_id: Unique customer identifier
                - annual_revenue: Customer annual revenue
                - debt_to_equity: Debt to equity ratio
                - payment_history_score: Historical payment score (0-100)
                
        Returns:
            DataFrame with predictions:
                - customer_id: Customer identifier
                - default_probability: Probability of default (0-1)
                - risk_category: Risk category (low/medium/high)
                - prediction_date: Timestamp of prediction
                
        Raises:
            FeatureEngineeringError: If required features cannot be computed
            ModelInferenceError: If model prediction fails
            
        Example:
            >>> model = FinancialRiskModel()
            >>> predictions = model.predict_default_probability(customer_df)
            >>> high_risk = predictions.filter(col("risk_category") == "high")
        """
        pass
```

These AI code standards ensure that machine learning implementations on the Keyera Data Platform maintain the same high standards of quality, maintainability, and integration as existing platform components while addressing the unique requirements of AI/ML workloads.
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IvanWeissVanDerPol) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
