## phishing-detector

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Run Development Server
```bash
source venv/bin/activate
uvicorn src.api.main:app --reload --host 127.0.0.1 --port 8000
```

### Run Tests
```bash
python -m pytest tests/ -v                    # All tests
python -m pytest tests/test_endpoints.py -v   # Specific test file
python tests/test_with_real_urls.py           # Real URL classification tests
```

### Train ML Models
```bash
python src/train.py \
  --legitimate data/raw/legitimate.csv \
  --phishing data/raw/phishing.csv \
  --output data/processed \
  --models xgboost rf gb \
  --workers 5 \
  --batch-size 100
```

### Test Credentials
- Username: `testuser` / Password: `TestPassword123!`

## Architecture

### ML Classification Pipeline
```
URL → Content Fetching (aiohttp/BeautifulSoup) → Feature Extraction
→ Feature Pipeline (Imputer + StandardScaler) → Ensemble Classifier
→ Threat Intelligence (VirusTotal) → Risk Scoring → Classification
```

The ensemble classifier (`src/models/ensemble_classifier.py`) uses a VotingClassifier combining XGBoost, Random Forest, and Gradient Boosting models. Models are loaded from pickle files in `data/processed/models/`.

### API Structure
- `src/api/main.py` - All FastAPI endpoints (authentication, classification, admin)
- `src/api/auth.py` - JWT authentication with 30-minute expiry, OAuth2PasswordBearer
- `src/api/database.py` - MongoDB async operations via Motor driver
- `src/predict.py` - Orchestrates ML predictions with threat intelligence integration

### Authentication Flow
1. Login validates password against bcrypt hash in MongoDB
2. JWT token generated and stored in cookie (`access_token`)
3. Protected endpoints extract token via `get_user_from_cookie()` or Authorization header
4. Admin endpoints check `is_admin` flag for role-based access

### Key Endpoints
- `POST /classify` - Single URL analysis (public)
- `POST /classify-batch` - Batch processing (requires Bearer token in Authorization header)
- `GET /scan-history` - User's scan history (requires Authorization header)
- `GET /api/admin/dashboard` - Admin analytics (cookie-based auth, requires is_admin)

### Database (MongoDB)
Collections: `users`, `scan_history`
- Connection configured via `MONGO_URL` environment variable
- Indexes created on startup: username (unique), email (unique), (user_id, timestamp)

### Feature Extraction
- `src/features/content_features.py` - HTML analysis: forms, scripts, iframes, DOM structure
- `src/features/enhanced_url_features.py` - URL analysis: length, HTTPS, subdomains, IP detection, suspicious patterns

## Environment Variables
Required in `.env`:
- `MONGO_URL` - MongoDB connection string
- `DB_NAME` - Database name (default: phishing_detector)
- `VIRUSTOTAL_API_KEY` - For threat intelligence lookups
- `GOOGLE_SAFE_BROWSING_API_KEY` - Optional

## Pydantic v2 Compatibility
The codebase uses Pydantic v2. Custom types like `PyObjectId` in `src/api/utils.py` use `__get_pydantic_core_schema__` instead of the deprecated `__get_validators__`.

## Model Version Compatibility
Models were trained with scikit-learn 1.2.2. Current environment may show `InconsistentVersionWarning` when loading pickled models. If models fail to load, retrain using `src/train.py`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ovlachi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ovlachi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
