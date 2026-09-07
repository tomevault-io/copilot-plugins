## entelechy

> The **Heuristic Engine V3.1** is a FastAPI-based web application that analyzes text input sequences, calculates confidence scores, and generates structural analysis reports. The system integrates a web frontend with a Python backend powered by machine learning utilities.

# Heuristic Engine: API Development Guide

## Project Overview
The **Heuristic Engine V3.1** is a FastAPI-based web application that analyzes text input sequences, calculates confidence scores, and generates structural analysis reports. The system integrates a web frontend with a Python backend powered by machine learning utilities.

**Tech Stack**: FastAPI, Uvicorn, Pydantic, Pandas, Scikit-learn

## Core API Architecture

### API Entry Points
- **GET `/`**: Serves the HTML dashboard UI (embedded in `api.py` as `UI_HTML`)
- **POST `/explain`**: Main analysis endpoint
  - **Request**: `RequestData` model with `text: str`
  - **Response**: `{"result": str, "confidence": float}`
  - **Backend**: Routes to `brain.process_concept(text)` from `brain.py`

### Data Contract & Serialization
The API endpoint expects `brain.process_concept()` to **always return a dictionary** with these exact fields:
```python
{
    "text": str,  # Analysis result text
    "confidence": float  # Confidence percentage (e.g., 95.5)
}
```

**Critical**: Any deviation from this contract (missing keys, wrong types) will cause serialization failures and 500 errors. See [current_status.md](handoff/current_status.md#current-status) for known issues.

## Key Files & Responsibilities

| File | Purpose | API Role |
|------|---------|----------|
| `api.py` | FastAPI app, request routing, UI serving | Entry point for all requests; orchestrates `/explain` logic |
| `brain.py` | Core analysis logic via `process_concept()` | Must return `{"text": str, "confidence": float}` |
| `requirements.txt` | Python dependencies | Ensure FastAPI, Uvicorn versions are pinned for Render deployment |
| `handoff/` | Documentation directory | See [technical_spec.md](handoff/technical_spec.md) for deployment info |

## Common Pitfalls & Debugging

1. **500 Internal Server Error on POST `/explain`**
   - **Cause**: `process_concept()` return value doesn't match expected schema
   - **Fix**: Verify `brain.py` returns exactly `{"text": str, "confidence": float}`
   - **Debug**: Add error handling in `api.py` `/explain` route; log exception details

2. **JSON Serialization Failures**
   - Ensure confidence scores are floats/ints (not strings or numpy types)
   - Verify no circular references or unserializable objects in the return dict

3. **Deployment Caching Issues**
   - Render.com may cache old versions; purge deployment cache when updating `brain.py`
   - Pin all dependency versions in `requirements.txt` for reproducibility

## Development Workflow

### Running Locally
```bash
python -m pip install -r requirements.txt
python api.py  # Runs on http://localhost:10000
```

### Testing the API
```bash
curl -X POST http://localhost:10000/explain \
  -H "Content-Type: application/json" \
  -d '{"text": "test input"}'
```

### Adding New Endpoints
1. Define Pydantic model for request data
2. Add `@app.post()` or `@app.get()` route
3. Return JSON-serializable dict
4. Update frontend `fetch()` calls if needed

## Related Documentation
- [Project Summary](handoff/project_summary.md)
- [Technical Specifications](handoff/technical_spec.md)
- [Action Items](handoff/action_items.md)
- [Current Status](handoff/current_status.md)

---
> Source: [joshuageoschool2020-cmyk/Entelechy](https://github.com/joshuageoschool2020-cmyk/Entelechy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
