## prompts

> Windsurf rules for Python data science with Scikit-learn, TensorFlow, and Jupyter.


# Data Science (Python, Scikit-learn, TensorFlow) Rules

## Python Environment & Setup

<python_environment>
- Use Python 3.9+ for compatibility with latest ML libraries
- Use virtual environments (venv) or conda for dependency management
- Pin package versions in requirements.txt for reproducibility
- Use pyproject.toml for modern Python project configuration
- Install Jupyter Lab for interactive development
- Example requirements.txt:
  ```
  numpy==1.24.3
  pandas==2.0.3
  scikit-learn==1.3.0
  tensorflow==2.13.0
  matplotlib==3.7.1
  seaborn==0.12.2
  jupyter==1.0.0
  ```
</python_environment>

<project_structure>
- Use consistent project structure following cookiecutter-data-science
- Organize code: `/src`, `/data`, `/notebooks`, `/models`, `/reports`
- Use config files for hyperparameters and settings
- Implement proper logging throughout the pipeline
- Example structure:
  ```
  project/
  ├── data/
  │   ├── raw/
  │   ├── processed/
  │   └── external/
  ├── src/
  │   ├── data/
  │   ├── features/
  │   ├── models/
  │   └── visualization/
  ├── notebooks/
  ├── models/
  └── reports/
  ```
</project_structure>

## Data Processing & Analysis

<data_processing>
- Use pandas for data manipulation and analysis
- Implement proper data validation and quality checks
- Handle missing values explicitly with strategies
- Use vectorized operations over loops for performance
- Implement data preprocessing pipelines
- Example data processing:
  ```python
  import pandas as pd
  import numpy as np
  
  def clean_data(df):
      # Handle missing values
      df = df.dropna(subset=['target'])
      df['feature1'] = df['feature1'].fillna(df['feature1'].median())
      
      # Remove outliers using IQR method
      Q1 = df['feature1'].quantile(0.25)
      Q3 = df['feature1'].quantile(0.75)
      IQR = Q3 - Q1
      df = df[~((df['feature1'] < (Q1 - 1.5 * IQR)) | 
                (df['feature1'] > (Q3 + 1.5 * IQR)))]
      
      return df
  ```
</data_processing>

<exploratory_analysis>
- Use descriptive statistics for initial data understanding
- Create visualizations with matplotlib and seaborn
- Implement correlation analysis and feature importance
- Use statistical tests for hypothesis validation
- Document findings in Jupyter notebooks
- Example EDA:
  ```python
  import matplotlib.pyplot as plt
  import seaborn as sns
  
  # Distribution analysis
  plt.figure(figsize=(12, 8))
  plt.subplot(2, 2, 1)
  sns.histplot(df['target'], bins=30)
  plt.title('Target Distribution')
  
  # Correlation heatmap
  plt.subplot(2, 2, 2)
  sns.heatmap(df.corr(), annot=True, cmap='coolwarm')
  plt.title('Feature Correlations')
  
  # Feature importance
  plt.subplot(2, 2, 3)
  feature_importance = df.corr()['target'].abs().sort_values(ascending=False)
  sns.barplot(x=feature_importance.values, y=feature_importance.index)
  plt.title('Feature Importance')
  ```
</exploratory_analysis>

## Machine Learning with Scikit-learn

<sklearn_practices>
- Use scikit-learn pipelines for preprocessing and modeling
- Implement proper train/validation/test splits
- Use cross-validation for model evaluation
- Implement feature scaling and encoding within pipelines
- Use grid search or random search for hyperparameter tuning
- Example pipeline:
  ```python
  from sklearn.pipeline import Pipeline
  from sklearn.preprocessing import StandardScaler, OneHotEncoder
  from sklearn.compose import ColumnTransformer
  from sklearn.ensemble import RandomForestClassifier
  from sklearn.model_selection import GridSearchCV
  
  # Preprocessing pipeline
  numeric_features = ['age', 'income']
  categorical_features = ['category', 'region']
  
  preprocessor = ColumnTransformer(
      transformers=[
          ('num', StandardScaler(), numeric_features),
          ('cat', OneHotEncoder(drop='first'), categorical_features)
      ]
  )
  
  # Model pipeline
  pipeline = Pipeline([
      ('preprocessor', preprocessor),
      ('classifier', RandomForestClassifier(random_state=42))
  ])
  
  # Hyperparameter tuning
  param_grid = {
      'classifier__n_estimators': [100, 200, 300],
      'classifier__max_depth': [10, 20, None]
  }
  
  grid_search = GridSearchCV(pipeline, param_grid, cv=5, scoring='accuracy')
  grid_search.fit(X_train, y_train)
  ```
</sklearn_practices>

<model_evaluation>
- Use appropriate metrics for classification and regression
- Implement cross-validation for robust evaluation
- Create confusion matrices and classification reports
- Use learning curves to diagnose bias/variance
- Implement feature importance analysis
- Example evaluation:
  ```python
  from sklearn.metrics import classification_report, confusion_matrix
  from sklearn.metrics import roc_auc_score, precision_recall_curve
  
  def evaluate_model(model, X_test, y_test):
      y_pred = model.predict(X_test)
      y_prob = model.predict_proba(X_test)[:, 1]
      
      print("Classification Report:")
      print(classification_report(y_test, y_pred))
      
      print(f"ROC AUC Score: {roc_auc_score(y_test, y_prob):.4f}")
      
      # Confusion matrix
      cm = confusion_matrix(y_test, y_pred)
      sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')
      plt.title('Confusion Matrix')
      plt.show()
  ```
</model_evaluation>

## Deep Learning with TensorFlow

<tensorflow_practices>
- Use TensorFlow 2.x with Keras API for model building
- Implement proper data preprocessing with tf.data
- Use callbacks for training optimization
- Implement model checkpointing and early stopping
- Use TensorBoard for experiment tracking
- Example neural network:
  ```python
  import tensorflow as tf
  from tensorflow.keras import layers, models, callbacks
  
  def create_model(input_shape, num_classes):
      model = models.Sequential([
          layers.Dense(128, activation='relu', input_shape=input_shape),
          layers.Dropout(0.3),
          layers.Dense(64, activation='relu'),
          layers.Dropout(0.3),
          layers.Dense(num_classes, activation='softmax')
      ])
      
      model.compile(
          optimizer='adam',
          loss='sparse_categorical_crossentropy',
          metrics=['accuracy']
      )
      
      return model
  
  # Training with callbacks
  model = create_model((X_train.shape[1],), len(np.unique(y_train)))
  
  callbacks_list = [
      callbacks.EarlyStopping(patience=10, restore_best_weights=True),
      callbacks.ReduceLROnPlateau(factor=0.5, patience=5),
      callbacks.ModelCheckpoint('best_model.h5', save_best_only=True)
  ]
  
  history = model.fit(
      X_train, y_train,
      epochs=100,
      batch_size=32,
      validation_split=0.2,
      callbacks=callbacks_list,
      verbose=1
  )
  ```
</tensorflow_practices>

<data_pipeline>
- Use tf.data for efficient data loading and preprocessing
- Implement proper data augmentation for image data
- Use prefetching and caching for performance
- Implement batch processing for large datasets
- Example data pipeline:
  ```python
  def create_dataset(X, y, batch_size=32, shuffle=True):
      dataset = tf.data.Dataset.from_tensor_slices((X, y))
      
      if shuffle:
          dataset = dataset.shuffle(buffer_size=1000)
      
      dataset = dataset.batch(batch_size)
      dataset = dataset.prefetch(tf.data.AUTOTUNE)
      
      return dataset
  
  # For image data
  def preprocess_image(image, label):
      image = tf.cast(image, tf.float32) / 255.0
      image = tf.image.resize(image, [224, 224])
      return image, label
  
  train_dataset = tf.data.Dataset.from_tensor_slices((X_train, y_train))
  train_dataset = train_dataset.map(preprocess_image)
  train_dataset = train_dataset.batch(32).prefetch(tf.data.AUTOTUNE)
  ```
</data_pipeline>

## Model Deployment & MLOps

<model_deployment>
- Use joblib for scikit-learn model serialization
- Save TensorFlow models in SavedModel format
- Implement model versioning and tracking
- Use Docker for containerized deployments
- Implement model serving with FastAPI or Flask
- Example model serving:
  ```python
  from fastapi import FastAPI
  import joblib
  import numpy as np
  
  app = FastAPI()
  model = joblib.load('model.pkl')
  
  @app.post("/predict")
  async def predict(features: dict):
      # Convert features to numpy array
      X = np.array(list(features.values())).reshape(1, -1)
      
      # Make prediction
      prediction = model.predict(X)[0]
      probability = model.predict_proba(X)[0].max()
      
      return {
          "prediction": int(prediction),
          "probability": float(probability)
      }
  ```
</model_deployment>

<experiment_tracking>
- Use MLflow or Weights & Biases for experiment tracking
- Log hyperparameters, metrics, and artifacts
- Implement model registry for production models
- Use version control for data and models (DVC)
- Example MLflow usage:
  ```python
  import mlflow
  import mlflow.sklearn
  
  with mlflow.start_run():
      # Log parameters
      mlflow.log_param("n_estimators", 100)
      mlflow.log_param("max_depth", 10)
      
      # Train model
      model.fit(X_train, y_train)
      
      # Log metrics
      accuracy = model.score(X_test, y_test)
      mlflow.log_metric("accuracy", accuracy)
      
      # Log model
      mlflow.sklearn.log_model(model, "model")
  ```
</experiment_tracking>

## Best Practices & Code Quality

<code_quality>
- Use type hints for better code documentation
- Implement proper error handling and logging
- Write unit tests for data processing functions
- Use docstrings for function documentation
- Follow PEP 8 style guidelines
- Use black for code formatting
- Example function with best practices:
  ```python
  import logging
  from typing import Tuple, Optional
  
  def split_data(
      X: np.ndarray, 
      y: np.ndarray, 
      test_size: float = 0.2,
      random_state: Optional[int] = None
  ) -> Tuple[np.ndarray, np.ndarray, np.ndarray, np.ndarray]:
      """
      Split data into training and testing sets.
      
      Args:
          X: Features array
          y: Target array
          test_size: Proportion of data for testing
          random_state: Random seed for reproducibility
          
      Returns:
          Tuple of (X_train, X_test, y_train, y_test)
      """
      try:
          from sklearn.model_selection import train_test_split
          
          return train_test_split(
              X, y, test_size=test_size, random_state=random_state
          )
      except Exception as e:
          logging.error(f"Error splitting data: {e}")
          raise
  ```
</code_quality>

<performance_optimization>
- Use numpy vectorization over Python loops
- Implement parallel processing with multiprocessing
- Use GPU acceleration for TensorFlow models
- Optimize memory usage with data types
- Profile code with cProfile for bottlenecks
- Use efficient data structures (pandas categorical)
</performance_optimization>

## Testing & Validation

<testing_practices>
- Write unit tests for data processing functions
- Test model performance on holdout datasets
- Implement data validation tests
- Use pytest for testing framework
- Test model robustness with edge cases
- Example test:
  ```python
  import pytest
  import numpy as np
  
  def test_data_preprocessing():
      # Test data
      X = np.array([[1, 2], [3, 4], [np.nan, 6]])
      
      # Process data
      X_processed = preprocess_data(X)
      
      # Assertions
      assert not np.isnan(X_processed).any()
      assert X_processed.shape == (3, 2)
      assert X_processed.dtype == np.float64
  ```
</testing_practices>

## References
- Scikit-learn Documentation: https://scikit-learn.org/stable/
- TensorFlow Documentation: https://www.tensorflow.org/guide
- Pandas Documentation: https://pandas.pydata.org/docs/
- MLflow Documentation: https://mlflow.org/docs/latest/index.html
- Python Data Science Handbook: https://jakevdp.github.io/PythonDataScienceHandbook/
- Cookiecutter Data Science: https://drivendata.github.io/cookiecutter-data-science/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/funsaized) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
