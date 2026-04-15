## raptor-project

> description: "Always quote rules being applied and explicitly state them in output"

# RAPTOR System Cursor Rules
version: 1.1

# Rule enforcement configuration
rules:
  enforce_quotes:
    description: "Always quote rules being applied and explicitly state them in output"
    enabled: true
    message: "Please quote the specific rule being applied and explain its impact"

  document_changes:
    description: "Update instructions.md when significant code changes are made"
    enabled: true
    pattern: "*.py"
    message: "Significant code changes detected. Update instructions.md accordingly"

  error_handling:
    description: "Enforce standardized error handling across all components"
    enabled: true
    pattern: "*.py"
    message: "Use error_handler.py for consistent error management"

  code_quality:
    description: "Enforce code quality standards"
    enabled: true
    requirements:
      - type_hints
      - docstrings
      - unit_tests
      - logging

# Action Plan Steps
action_plan:
  phase_1_cleanup:
    priority: 1
    tasks:
      - "Standardize error handling across all components"
      - "Remove debug file operations and temporary storage"
      - "Implement proper logging in all modules"
      - "Clean up namespace references in Pinecone operations"

  phase_2_stability:
    priority: 2
    tasks:
      - "Implement robust chunking with size validation"
      - "Add HDBSCAN clustering parameters validation"
      - "Enhance tree building error detection"
      - "Implement automatic recovery mechanisms"

  phase_3_monitoring:
    priority: 3
    tasks:
      - "Set up comprehensive system monitoring"
      - "Implement performance metrics collection"
      - "Create monitoring dashboards"
      - "Add automated alerts for critical issues"

  phase_4_optimization:
    priority: 4
    tasks:
      - "Optimize embedding batch processing"
      - "Improve clustering performance"
      - "Enhance visualization rendering"
      - "Implement caching mechanisms"

# Directory structure enforcement
directories:
  src:
    - analysis
    - clustering
    - embedding
    - monitoring
    - prepare
    - scripts
    - storage
    - summarization
    - tree
    - update
    - utils
    - visualization

  data:
    pattern: "{index_name}/"
    subdirs:
      - raw
      - processed
      - failed
      - archive

  test_outputs:
    - tree_viz
    - analysis
    - reports

# Protected files requiring special attention
protected_files:
  - src/tree/tree_manager.py
  - src/tree/raptor_tree.py
  - src/storage/pinecone_manager.py
  - src/clustering/**/*.py
  - src/embedding/**/*.py
  - src/summarization/**/*.py
  - src/monitoring/**/*.py
  - src/utils/error_handler.py
  - src/utils/config.py

# Code modification rules
modification_rules:
  protected_components:
    requires:
      - feature_branch
      - test_coverage
      - code_review
      - backup
      - documentation
      - error_handling_review
      - performance_impact_assessment

  code_standards:
    requires:
      - type_hints
      - docstrings
      - error_handling
      - logging
      - unit_tests
      - integration_tests

# Performance requirements
performance:
  embedding:
    batch_size: 100
    max_retries: 3
    timeout: 30

  clustering:
    min_cluster_size: 5
    min_samples: 3
    max_cluster_size: 1000

  visualization:
    max_nodes_per_level: 1000
    cache_timeout: 3600

# Monitoring configuration
monitoring:
  metrics:
    - processing_time
    - error_rate
    - memory_usage
    - api_latency
    - cluster_sizes
    - embedding_quality

  alerts:
    error_threshold: 0.05
    latency_threshold: 5000
    memory_threshold: 0.85

# Testing requirements
testing:
  coverage:
    minimum: 80
    critical_paths: 95
  
  required_tests:
    - unit
    - integration
    - performance
    - recovery

# Documentation requirements
documentation:
  required_sections:
    - Overview
    - Key Components
    - System Architecture
    - Data Structures
    - Directory Structure
    - Implementation Details
    - System Monitoring
    - Best Practices
    - Troubleshooting Guide
    - Analysis Reports
    - API Documentation
    - Deployment Guide
    - Recovery Procedures
    - Performance Optimization

# Error handling requirements
error_handling:
  required_components:
    - error_logging
    - retry_mechanism
    - fallback_strategy
    - user_notification
    - error_reporting

# Integration requirements
integrations:
  pinecone:
    singleton: true
    batch_size: 100
    timeout: 30
    retry_attempts: 3
  
  openai:
    models:
      embedding: "text-embedding-3-small"
      summarization: "gpt-4-turbo-preview"
    require_token_tracking: true
    max_tokens:
      chunk_size: 8000
      summary_size: 4000

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slightlyarrogant) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
