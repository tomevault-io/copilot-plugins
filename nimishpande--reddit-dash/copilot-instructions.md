## reddit-dash

> These rules govern the extraction, mapping, and aggregation of ingredient data from multiple sources for the comprehensive cosmetic ingredient CSV. They ensure each column is populated with the most authoritative and relevant data, with clear fallbacks and robust logic.

# Cursor Rules for Ingredient Data Extraction and CSV Mapping

## Overview
These rules govern the extraction, mapping, and aggregation of ingredient data from multiple sources for the comprehensive cosmetic ingredient CSV. They ensure each column is populated with the most authoritative and relevant data, with clear fallbacks and robust logic.

---

## 1. Source Prioritization
- INCI Database (`separated_ingredients/`): Use for all technical, chemical, and standardized fields (e.g., INCI name, CAS number, classification, function, usage %, safety rating).
- Comprehensive Reports (`reddit analysis/`): Use for sentiment, dosage, product forms, summary statistics, and evidence-based narrative. Extract pros/cons, controversies, and safety notes from here first.
- Reddit Comments (`raw data/`): Use for real-world user experiences, effectiveness, study mentions, and to supplement/validate sentiment and counts. Use as fallback for community sentiment and anecdotal evidence.

---

## 2. Column-by-Column Mapping
- Ingredient Name: INCI name from INCI Database. Fallback: common name from INCI Database.
- Research: Count of Reddit posts/comments, and summary of any scientific studies or trials mentioned (from Reddit and reports).
- %usage: Extract from INCI Database. Fallback: extract from report using regex/NER for percentages.
- Pros: Extract positive effects/benefits from Comprehensive Report. Supplement with Reddit comments using sentiment/keyword extraction.
- Cons: Extract negative effects/risks from Comprehensive Report. Supplement with Reddit comments using sentiment/keyword extraction.
- Effectiveness: Extract statements about efficacy from Reddit comments (using effectiveness keywords and sentiment). Fallback: summary report.
- Reddit: Aggregate overall sentiment from Reddit comments (VADER/TextBlob). Use average compound score to classify as Positive/Neutral/Negative.
- Natural or synthesised: Use classification/origin from INCI Database. Fallback: extract from report.

---

## 3. Aggregation and Summarization Logic
- Always use the most authoritative source first for each field, with clear fallback to the next best source if missing.
- For sentiment fields (Pros/Cons/Effectiveness/Reddit), use both keyword and sentiment analysis. Aggregate multiple sentences, truncate for CSV if needed.
- For research/study mentions, extract up to 3 most relevant study references from Reddit and reports.
- For technical fields, never use Reddit as primary source—only as validation or supplement.
- For all fields, ensure robust handling of missing or malformed data (skip, log, or use fallback as appropriate).

---

## 4. Output Formatting
- Output must match the sample CSV structure exactly (column order, naming, and types).
- Truncate long text fields to fit CSV constraints (e.g., 30-50 words for Pros/Cons/Effectiveness).
- For each ingredient, provide a comprehensive LLM-generated summary using a prompt that combines technical, report, and Reddit data, clearly distinguishing between evidence types.

---

## 5. General Principles
- Maintain a neutral, evidence-based tone in all summaries.
- Clearly distinguish between verified facts (technical/report) and anecdotal/community data (Reddit).
- Log all extraction steps and fallbacks for traceability.
- Update these rules as new sources or requirements emerge.


# Essential Project Rules - Starter Template

## Code Style
- Use descriptive variable names (no `x`, `data`, `temp`)
- Keep functions under 20 lines when possible
- Use early returns to reduce nesting
- Add type hints for function parameters and return values
- Include docstrings for all functions

Example function structure:
```python
def process_user_data(user_id: str, options: dict) -> Optional[dict]:
    """
    Process user data with given options.
    
    Args:
        user_id: Unique identifier for user
        options: Processing configuration dict
        
    Returns:
        Processed data dict or None if processing fails
    """
```

## Error Handling
- Always use specific exception types, never bare `except:`
- Include helpful error messages with context
- Log errors before handling them
- Use try-catch for file operations, API calls, and data parsing

Required error handling pattern:
```python
try:
    result = risky_operation()
except SpecificError as e:
    logger.error(f"Operation failed for {context}: {e}")
    return None
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    raise
```

## Security
- Never hardcode API keys, passwords, or secrets
- Use environment variables for sensitive data
- Validate all user inputs
- Use parameterized queries for databases

## Testing
- Write tests for all public functions
- Include edge cases (empty inputs, invalid data)
- Use descriptive test names that explain the scenario
- Mock external dependencies (APIs, files, databases)

## Documentation
- Write self-explaining code that needs minimal comments
- Add comments only to explain WHY, not WHAT
- Keep README.md updated with setup instructions
- Document any complex algorithms or business logic

## File Organization
- Group related functions in the same module
- Use consistent naming for files and folders
- Separate configuration from business logic
- Create a clear project structure

## Git Practices
- Write clear commit messages
- Make small, focused commits
- Review your changes before committing
- Don't commit debug code or temporary files

## Performance
- Use appropriate data structures for the task
- Close files and database connections properly
- Use context managers (`with` statements)
- Cache expensive operations when it makes sense

## Required Imports Structure
Organize imports in this order:
```python
# Standard library imports
import os
import json
from typing import Optional, Dict, List

# Third-party imports
import pandas as pd
import requests

# Local imports
from .utils import helper_function
```

## Logging
Always include logging setup:
```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Use in functions
logger.info("Starting processing")
logger.error(f"Failed to process {item}: {error}")
```

## Progress Tracking
For long-running operations, include progress indicators:
- "🔍 Processing {item_name}..."
- "✅ Completed successfully"
- "❌ Error: {specific_error_message}"
- "🎉 Finished! Processed {count} items"

## Configuration
- Use environment variables for settings that change between environments
- Create a config.py file for application settings
- Document all required environment variables
- Provide example .env files

## Code Review Checklist
Before submitting code, verify:
- [ ] Function names clearly describe what they do
- [ ] Error handling covers expected failure cases
- [ ] No hardcoded values that should be configurable
- [ ] Logging provides useful information for debugging
- [ ] Code follows the established patterns in the project
- [ ] Tests cover the main functionality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nimishpande) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
