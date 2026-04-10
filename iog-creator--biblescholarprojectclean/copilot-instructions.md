## biblescholarprojectclean

> description: Guidelines for optimizing contextual insights modules with DSPy teleprompters and handling integration issues

alwaysApply: false
description: Guidelines for optimizing contextual insights modules with DSPy teleprompters and handling integration issues
globs:
  - "scripts/*contextual_insights*.py"
  - "scripts/optimize_contextual_insights.py"
  - "src/dspy_programs/contextual_insights_program.py"
  - "train_contextual_insights.bat"
  - "test_contextual_insights.py"
title: Contextual Insights Optimization
type: feature
---

# Contextual Insights Optimization

## Overview

This rule documents the process and best practices for optimizing the Contextual Insights feature using DSPy teleprompters and handling integration issues with LM Studio.

## Training Data Generation

Training data must be generated for each insight module in JSONL format:

1. **Directory Structure**:
   ```
   data/processed/dspy_training_data/contextual_insights/
     ├── summary_examples.jsonl
     ├── cross_reference_examples.jsonl
     ├── theological_terms_examples.jsonl
     ├── historical_context_examples.jsonl
     ├── original_language_examples.jsonl
     └── related_entities_examples.jsonl
   ```

2. **Example Format**:
   Each example should be a JSON object with `input` and `output` fields, matching the module's signature:
   ```json
   {"input": {"verse_reference": "John 3:16", "verse_text": "For God so loved..."}, "output": {"summary_text": "This verse explains..."}}
   ```

3. **Required Scripts**:
   - `scripts/generate_contextual_insights_training.py`: Generates training examples
   - `scripts/optimize_contextual_insights.py`: Runs optimization using DSPy teleprompters
   - `scripts/load_optimized_contextual_insights.py`: Loads optimized modules into the main program

## Known Issues and Solutions

### LM Studio + DSPy JSON Schema Compatibility

1. **Issue**: LM Studio requires `response_format.type` to be `json_object` while DSPy expects `json_schema`
   
2. **Solution**:
   - Use the `dspy_json_patch.py` to patch the DSPy module for proper JSON schema handling
   - Always apply the patch before initializing any DSPy components:
     ```python
     import dspy
     import dspy_json_patch
     dspy_json_patch.apply_patches()
     dspy.settings.experimental = True
     ```
   - For Contextual Insights, this is applied in both the API and optimization scripts

3. **When Optimizing**:
   - Use simple optimizers like `BootstrapFewShot` as they are more compatible with LM Studio
   - Set proper Example inputs using `Example(input={...}, output={...})` format
   - Provide clear input/output field types in the signature classes

### MLflow Integration

1. **Make MLflow Optional**:
   The optimization scripts should work even if MLflow is not available:

   ```python
   try:
       import mlflow
       mlflow_available = True
       # Configure MLflow
   except (ImportError, Exception) as e:
       mlflow_available = False
       logger.warning(f"MLflow not available: {e}")
   ```

2. **Fallback Pattern**:
   ```python
   def log_metrics(metrics):
       if mlflow_available:
           mlflow.log_metrics(metrics)
       else:
           # Save metrics to a local file instead
           with open("optimization_metrics.json", "w") as f:
               json.dump(metrics, f)
   ```

## Optimization Process

1. **Generate Training Data**:
   ```bash
   python scripts/generate_contextual_insights_training.py
   ```

2. **Optimize Each Module**:
   ```bash
   python scripts/optimize_contextual_insights.py --module summary --optimizer bootstrap
   ```

3. **Load Optimized Modules**:
   ```python
   # In contextual_insights_program.py
   from scripts.load_optimized_contextual_insights import get_optimized_modules
   
   optimized_modules = get_optimized_modules()
   if optimized_modules.get('summary_generator'):
       self.generate_summary = optimized_modules['summary_generator']
   ```

## Testing

1. **Program-Only Testing**:
   Test just the DSPy program components without API integration:
   ```bash
   python test_contextual_insights.py --program-only
   ```

2. **API Testing**:
   Test the full API integration:
   ```bash
   python test_contextual_insights.py
   ```

3. **Web Server**:
   Start the web server with:
   ```bash
   python run_contextual_insights_web.py
   # or
   run_contextual_insights_web.bat
   ```

4. **API Health Check**:
   ```bash
   curl http://localhost:5001/api/contextual_insights/health
   ```

5. **API Request Test**:
   ```bash
   curl -X POST http://localhost:5001/api/contextual_insights/insights \
     -H "Content-Type: application/json" \
     -d '{"type":"verse","reference":"John 3:16","translation":"KJV"}'
   ```

## Common Errors and Fixes

1. **Original Language Notes Error**:
   ```
   Could not retrieve original language text: get_original_language_verse_text() missing 2 required positional arguments: 'chapter_num' and 'verse_num'
   ```
   
   **Fix**: Update the FocusProcessor to parse verse references into book, chapter, and verse components before calling get_original_language_verse_text().

2. **Blueprint Error**:
   ```
   AttributeError: 'Blueprint' object has no attribute 'before_app_first_request'
   ```
   
   **Fix**: Replace `@api_blueprint.before_app_first_request` with `@api_blueprint.before_app_request` and add logic to only initialize once.

3. **Example Format Errors**:
   ```
   AttributeError: 'dict' object has no attribute 'with_inputs'
   ```
   
   **Fix**: Create DSPy Examples with explicit fields matching signature requirements.

## Documentation Updates

After making changes to the optimization process:

1. Update `docs/features/contextual_insights.md` with latest implementation details
2. Update `docs/roadmaps/contextual_insights_feature_roadmap.md` to reflect current status
3. Document any API changes in `docs/reference/API_REFERENCE.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iog-creator) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
