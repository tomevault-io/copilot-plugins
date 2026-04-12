## base-mcp-server

> AI Development Rules and Best Practices

# AI Development Rules and Best Practices

This file contains rules for AI-related code generation. It guides the AI in generating code that adheres to best practices in AI development, machine learning, and intelligent system design.

## Core AI Development Principles

### Data Management and Ethics
- Always implement data privacy and security measures (GDPR, CCPA compliance)
- Use anonymization and differential privacy techniques when handling sensitive data
- Implement proper data validation and sanitization
- Document data lineage and maintain audit trails
- Apply bias detection and mitigation strategies
- Ensure data quality through automated validation pipelines

### Model Development
- Follow MLOps best practices for model lifecycle management
- Implement proper train/validation/test splits with stratification
- Use cross-validation for robust model evaluation
- Apply feature engineering and selection systematically
- Implement proper hyperparameter tuning with automated search
- Use ensemble methods where appropriate for improved performance
- Document model assumptions, limitations, and performance metrics

### Code Structure and Quality
- Separate data processing, training, and inference code into distinct modules
- Use configuration files (YAML/JSON) for hyperparameters and settings
- Implement comprehensive logging and monitoring
- Follow reproducibility principles with seed management
- Use type hints and proper documentation for AI/ML functions
- Implement unit tests for data processing and model components

## Framework-Specific Guidelines

### TensorFlow/Keras
```python
# Use tf.data for efficient data pipelines
dataset = tf.data.Dataset.from_tensor_slices((X_train, y_train))
dataset = dataset.batch(batch_size).prefetch(tf.data.AUTOTUNE)

# Implement callbacks for training monitoring
callbacks = [
    tf.keras.callbacks.EarlyStopping(patience=10, restore_best_weights=True),
    tf.keras.callbacks.ReduceLROnPlateau(factor=0.5, patience=5),
    tf.keras.callbacks.ModelCheckpoint(filepath='best_model.h5', save_best_only=True)
]
```

### PyTorch
```python
# Use DataLoader for efficient batch processing
dataloader = torch.utils.data.DataLoader(
    dataset, batch_size=batch_size, shuffle=True, num_workers=4
)

# Implement proper device handling
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = model.to(device)
```

### Scikit-learn
```python
# Use pipelines for preprocessing and modeling
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('selector', SelectKBest(f_classif, k=10)),
    ('classifier', RandomForestClassifier(random_state=42))
])

# Implement proper cross-validation
scores = cross_val_score(pipeline, X, y, cv=5, scoring='accuracy')
```

## Model Deployment and Serving

### API Design
- Design RESTful APIs for model serving with proper versioning
- Implement request validation and error handling
- Use async/await for non-blocking operations
- Implement rate limiting and authentication
- Provide clear API documentation with examples

### Container and Cloud Deployment
```dockerfile
# Multi-stage Docker builds for AI applications
FROM python:3.9-slim as base
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM base as production
COPY src/ ./src/
COPY models/ ./models/
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Model Monitoring
- Implement data drift detection
- Monitor model performance metrics in production
- Set up alerts for performance degradation
- Track prediction confidence and uncertainty
- Log all predictions and outcomes for analysis

## Testing and Validation

### Unit Testing
```python
import pytest
import numpy as np
from src.preprocessing import normalize_features

def test_normalize_features():
    # Test data normalization
    data = np.array([[1, 2], [3, 4], [5, 6]])
    normalized = normalize_features(data)

    assert normalized.mean(axis=0).tolist() == pytest.approx([0, 0], abs=1e-10)
    assert normalized.std(axis=0).tolist() == pytest.approx([1, 1], abs=1e-10)
```

### Integration Testing
- Test complete ML pipelines end-to-end
- Validate model loading and inference
- Test API endpoints with various input scenarios
- Verify error handling and edge cases

## Security and Compliance

### Model Security
- Implement input validation to prevent adversarial attacks
- Use secure model serialization formats
- Protect model intellectual property
- Implement access controls for model endpoints
- Regular security audits and vulnerability assessments

### Privacy Protection
- Implement federated learning where applicable
- Use homomorphic encryption for sensitive computations
- Apply differential privacy to training data
- Ensure right-to-be-forgotten compliance

## Performance Optimization

### Computational Efficiency
- Profile code to identify bottlenecks
- Use vectorized operations instead of loops
- Implement caching for expensive computations
- Optimize memory usage with generators and lazy loading
- Use GPU acceleration where beneficial

### Scalability
- Design stateless inference services
- Implement horizontal scaling strategies
- Use message queues for asynchronous processing
- Implement database connection pooling
- Monitor resource utilization and auto-scaling

## Documentation and Maintenance

### Model Documentation
- Document model architecture and design decisions
- Maintain model cards with performance metrics
- Document training procedures and hyperparameters
- Keep track of model versions and changes
- Document known limitations and failure modes

### Code Documentation
```python
def train_model(X_train: np.ndarray, y_train: np.ndarray,
                config: Dict[str, Any]) -> Tuple[Any, Dict[str, float]]:
    """
    Train a machine learning model with the given data and configuration.

    Args:
        X_train: Training features with shape (n_samples, n_features)
        y_train: Training labels with shape (n_samples,)
        config: Configuration dictionary containing hyperparameters

    Returns:
        Tuple of (trained_model, metrics_dict)

    Raises:
        ValueError: If input data is invalid or config is missing required keys
    """
```

## Continuous Integration/Deployment

### CI/CD Pipeline
```yaml
# Example GitHub Actions workflow for ML projects
name: ML Pipeline
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.9
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests
        run: pytest tests/
      - name: Run model validation
        run: python scripts/validate_model.py
```

### Model Versioning
- Use semantic versioning for models
- Implement A/B testing for model deployments
- Maintain model registry with metadata
- Implement rollback mechanisms
- Track model lineage and dependencies

## Common Patterns and Anti-Patterns

### Best Practices
✅ Use configuration files for hyperparameters
✅ Implement proper error handling and logging
✅ Validate input data thoroughly
✅ Use cross-validation for model evaluation
✅ Implement monitoring and alerting
✅ Document model assumptions and limitations

### Anti-Patterns to Avoid
❌ Hardcoding hyperparameters in code
❌ Training on the entire dataset without validation
❌ Ignoring data leakage in feature engineering
❌ Deploying models without monitoring
❌ Not handling missing or invalid input data
❌ Overfitting to validation data

## Tools and Libraries Recommendations

### Essential Libraries
- **Data Processing**: pandas, numpy, dask
- **Machine Learning**: scikit-learn, xgboost, lightgbm
- **Deep Learning**: tensorflow, pytorch, transformers
- **Visualization**: matplotlib, seaborn, plotly
- **Experimentation**: mlflow, wandb, tensorboard
- **Testing**: pytest, hypothesis, great-expectations

### Infrastructure Tools
- **Containerization**: Docker, Kubernetes
- **Orchestration**: Airflow, Kubeflow, MLflow
- **Monitoring**: Prometheus, Grafana, ELK Stack
- **Cloud Platforms**: AWS SageMaker, Azure ML, GCP AI Platform

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michealmueller)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/michealmueller)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
