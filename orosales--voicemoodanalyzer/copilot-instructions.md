## voicemoodanalyzer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VoiceMoodAnalyzer is a multi-stage AI pipeline that analyzes emotional state from voice recordings by combining three signals:
1. Audio transcription (Whisper.cpp - local, small model ~466MB)
   - No internet required for transcription
   - 6x realtime speed, excellent accuracy
2. Audio emotion detection (Wav2Vec2 model - local, 97.5% accuracy)
   - Model: r-f/wav2vec-english-speech-emotion-recognition
   - 7 emotions: angry, disgust, fear, happy, neutral, sad, surprise
3. Text sentiment analysis (DistilRoBERTa model - local)
4. Emotion fusion via a database-driven matrix lookup

The system is fully containerized with Docker and designed for Azure VM deployment with mobile-friendly web access. All AI models run locally with no external API dependencies.

## Development Commands

### Docker Environment (Primary Development Method)

```bash
# Start all services (backend, frontend, postgres)
docker-compose up -d --build

# View logs from all containers
docker-compose logs -f

# View logs from specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f postgres

# Restart specific service
docker-compose restart backend

# Stop all services
docker-compose down

# Stop and remove volumes (full reset including database)
docker-compose down -v

# Rebuild after code changes
docker-compose up -d --build
```

### Local Development (Without Docker)

**Backend:**
```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Requires PostgreSQL running separately
# Models will download on first run (~2GB)
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev          # Development server on port 3000
npm run build        # Production build
npm run preview      # Preview production build
```

### Database Operations

```bash
# Access PostgreSQL shell
docker-compose exec postgres psql -U postgres -d mito_books

# Backup database
docker-compose exec postgres pg_dump -U postgres mito_books > backup.sql

# Restore database
docker-compose exec -T postgres psql -U postgres mito_books < backup.sql

# Reset database (re-run init scripts)
docker-compose down -v && docker-compose up -d
```

### Testing

```bash
# Test API endpoints
./test_api.sh

# Manual API test
curl http://localhost:8000/
curl http://localhost:8000/api/matrix
curl -X POST http://localhost:8000/api/analyze -F "file=@test.mp3"
```

## Architecture & Data Flow

### Request Pipeline (POST /api/analyze)

The analysis pipeline in `backend/app.py` follows this exact sequence:

1. **Upload & Validation** (`app.py:analyze_voice`)
   - Validate file size (<25MB) and format (.wav, .mp3, .m4a, .ogg, .flac, .webm)
   - Save to temporary file

2. **Whisper Transcription** (`services/whisper_cpp_service.py`)
   - Local transcription using whisper.cpp (small model)
   - No internet required, 6x realtime speed
   - Returns: `transcribed_text: str`

3. **Audio Emotion Detection** (`services/audio_emotion.py`)
   - **Duration-based processing**: Only runs for recordings ≤15 seconds
   - For recordings >15 seconds: Skipped (defaults to "neutral" with 0.0 confidence) to save processing time (~15-20s per analysis)
   - Load audio, resample to 16kHz mono
   - Run through Wav2Vec2 model (`r-f/wav2vec-english-speech-emotion-recognition`)
   - **97.5% accuracy** - Fine-tuned on 4,720 samples from SAVEE, RAVDESS, and TESS datasets
   - Returns: `audio_emotion: str` (angry/disgust/fear/happy/neutral/sad/surprise), `audio_confidence: float`

4. **Text Emotion Detection** (`services/text_emotion.py`)
   - Tokenize transcribed text
   - Run through DistilRoBERTa model (`j-hartmann/emotion-english-distilroberta-base`)
   - Returns: `text_emotion: str` (neutral/happy/sad/angry/surprised/fearful/disgusted), `text_confidence: float`

5. **Emotion Fusion** (`services/fusion_service.py`)
   - Database lookup in `voice_matrix` table using (audio_emotion, text_emotion) as composite key
   - Fallback strategy: neutral+neutral → ultimate fallback: "Unknown"
   - Returns: `{final_mood, emoji, description}`

6. **Database Persistence** (`models/voice_analysis.py`)
   - Save complete analysis to `voice_analysis` table
   - Includes all intermediate results + final mood

### ML Model Lifecycle

**Singleton Pattern for Models** (important for memory efficiency):
- `get_whisper_service()` in `whisper_cpp_service.py` - loads Whisper.cpp once, caches globally (lazy loaded)
- `get_audio_emotion_service()` in `audio_emotion.py` - loads Wav2Vec2 once, caches globally
- `get_text_emotion_service()` in `text_emotion.py` - loads DistilRoBERTa once, caches globally
- Hugging Face models preload on app startup via `@app.on_event("startup")` in `app.py:41-48`
- Whisper.cpp model loads on first transcription request (lazy loading)

**First Run Behavior**:
- Hugging Face models download to Docker container (~2GB)
- Whisper.cpp small model downloads on first use (~466MB)
- Total model size: ~2.5GB
- Initial build takes 10-15 minutes, subsequent starts <30 seconds

### Database Schema Details

**voice_matrix** - Fusion matrix (seeded from `db/init/02-seed-fusion-matrix.sql`)
- Composite unique key on (audio_emotion, text_emotion)
- 30+ pre-configured emotion combinations
- Used for lookup in `FusionService.get_final_mood()`

**voice_analysis** - Analysis history
- Created on every successful analysis
- Never deleted (append-only audit trail)
- Queried by `/api/history` endpoint

### Configuration System

**Environment Variables** (`.env` → `backend/core/config.py`):
- `POSTGRES_*` - Database connection (default: localhost:5436, user: postgres, pass: 123, db: mito_books)
- Config accessed via `get_settings()` singleton (cached with `@lru_cache`)
- **Note**: OPENAI_API_KEY is NO LONGER REQUIRED (using local whisper.cpp)

**Pydantic Settings**:
- `core/config.py` validates all environment variables on startup
- Generates `database_url` property for SQLAlchemy
- No external API keys needed - all models run locally

## Key Implementation Details

### Audio Processing

**Audio Emotion Service** (`services/audio_emotion.py`):
- **Model**: `r-f/wav2vec-english-speech-emotion-recognition` (Wav2Vec2 architecture)
- **Accuracy**: 97.5% on evaluation set
- **Training**: Fine-tuned on 4,720 samples from SAVEE, RAVDESS, and TESS datasets
- **Duration Threshold**: Only runs for recordings ≤15 seconds
  - Recordings >15 seconds: Audio emotion skipped, defaults to "neutral" (confidence: 0.0)
  - Reason: Performance optimization (audio emotion detection takes ~15-20s for long files)
  - Text emotion analysis still runs for all recordings regardless of length
- Requires exactly 16kHz mono audio for Wav2Vec2
- Automatic resampling: `torchaudio.transforms.Resample(orig_sr, 16000)`
- Stereo → mono: `torch.mean(waveform, dim=0, keepdim=True)`
- Returns 7 emotions: angry, disgust, fear, happy, neutral, sad, surprise
- Uses `Wav2Vec2FeatureExtractor` for preprocessing

**Text Emotion Service** (`services/text_emotion.py`):
- Supports 7 raw emotions from model: anger, disgust, fear, joy, neutral, sadness, surprise
- Maps to simplified labels via `emotion_mapping` dict
- Truncates to 512 tokens (DistilRoBERTa max length)
- Empty text fallback: returns ("neutral", 1.0)

### Frontend Architecture

**Component Hierarchy**:
```
App.tsx
├── AudioRecorder.tsx    - MediaRecorder API, real-time countdown
├── FileUploader.tsx     - File input with format validation
├── LoadingSpinner.tsx   - During analysis
└── MoodResult.tsx       - Display all results with confidence bars
```

**API Integration** (`services/api.ts`):
- Axios instance with 60s timeout (for long Whisper API calls)
- FormData upload for multipart/form-data
- Base URL: `/api` (proxied by nginx to backend:8000)

**State Management** (`App.tsx`):
- No Redux/Zustand - simple useState hooks
- Three states: `isAnalyzing`, `result`, `error`
- File/Blob → File object → `analyzeVoice()` → display result

### Nginx Configuration

**Critical Proxy Settings** (`frontend/nginx.conf`):
- `/api/*` proxied to `http://backend:8000/api/*`
- `client_max_body_size 25M` - matches backend upload limit
- Timeouts: 60s (connect/send/read) for Whisper API latency
- SPA routing: `try_files $uri $uri/ /index.html` for React Router compatibility

## Customization Points

### Adding New Emotions to Fusion Matrix

1. Edit `db/init/02-seed-fusion-matrix.sql`
2. Add INSERT statements with new (audio_emotion, text_emotion) combinations
3. Rebuild database: `docker-compose down -v && docker-compose up -d`

Example:
```sql
INSERT INTO voice_matrix (audio_emotion, text_emotion, final_mood, emoji, description) VALUES
('happy', 'excited', 'Extremely Enthusiastic', '🤩', 'High energy and excitement.');
```

### Current Audio Emotion Model (Upgraded)

**Model History**:
- **Previous**: `superb/hubert-base-superb-er` (HuBERT, ~60-70% accuracy, 4 emotions)
- **Current**: `r-f/wav2vec-english-speech-emotion-recognition` (Wav2Vec2, 97.5% accuracy, 7 emotions)
- **Upgrade Date**: November 2025
- **Reason**: Better accuracy for detecting subtle voice tone differences with same text

**Current Model Details**:
- **HuggingFace**: https://huggingface.co/r-f/wav2vec-english-speech-emotion-recognition
- **Base**: Fine-tuned from `jonatasgrosman/wav2vec2-large-xlsr-53-english`
- **Training Data**: 4,720 audio samples from:
  - SAVEE: 480 files (4 male actors)
  - RAVDESS: 1,440 files (24 professional actors)
  - TESS: 2,800 files (2 female actors)
- **Accuracy**: 97.5% on evaluation set (validation loss: 0.104)
- **Emotions**: angry, disgust, fear, happy, neutral, sad, surprise
- **Architecture**: Wav2Vec2ForSequenceClassification
- **License**: Apache 2.0

### Swapping ML Models

**Audio Model** (`services/audio_emotion.py`):
```python
# Change model_name in __init__
self.model_name = "ehcalabres/wav2vec2-lg-xlsr-en-speech-emotion-recognition"  # Alternative (82% accuracy, 8 emotions)
# Update emotion_labels to match new model's output classes
```

**Text Model** (`services/text_emotion.py`):
```python
# Change model_name in __init__
self.model_name = "cardiffnlp/twitter-roberta-base-emotion"  # Example
# Update emotion_labels and emotion_mapping
```

### Adjusting Upload Limits

Change in three places:
1. `backend/core/config.py` - `MAX_UPLOAD_SIZE = 25 * 1024 * 1024`
2. `frontend/nginx.conf` - `client_max_body_size 25M;`
3. `frontend/src/components/FileUploader.tsx` - Update UI text

## Database Connection Parameters

**Important**: The system uses an **existing PostgreSQL database on the host** (not in Docker):
- Host: `localhost` (local dev) or `host.docker.internal` (Docker containers)
- Port: `5436`
- Database: `mito_books`
- User: `postgres`
- Password: `123`

Connection string format: `postgresql://postgres:123@localhost:5436/mito_books`

**Database Initialization**: Since PostgreSQL is not managed by Docker Compose, you must manually create tables:
```bash
psql -h localhost -p 5436 -U postgres -d mito_books -f db/init/01-init-tables.sql
psql -h localhost -p 5436 -U postgres -d mito_books -f db/init/02-seed-fusion-matrix.sql
```

## Deployment Notes

### Azure VM Requirements
- Minimum: 4 vCPUs, 8GB RAM, 50GB disk
- Recommended: Standard_D4s_v3 or larger
- Open ports: 80 (HTTP), 443 (HTTPS), 22 (SSH)
- First startup downloads ML models (~2.5GB) - factor into deployment time
- **No internet required after initial setup** - all models run locally

### Production Checklist
1. Change `POSTGRES_PASSWORD` in `.env`
2. Set `allow_origins` in `app.py` CORS to specific domain (not `["*"]`)
3. Add SSL certificate (Let's Encrypt) and update nginx.conf
4. Set up systemd service for auto-start (see `DEPLOYMENT.md`)
5. Configure firewall (UFW)
6. Enable Docker BuildKit for faster builds
7. Set up database backups (pg_dump cron job)

### Environment-Specific Behavior
- Hugging Face models load on startup - increases container start time
- Whisper.cpp model loads on first transcription (lazy loading)
- No internet connection required for inference - all models run locally
- PostgreSQL init scripts run only on first volume creation
- Frontend build happens in Dockerfile (not at runtime)

## Common Troubleshooting

### "Models downloading slowly"
- First run downloads ~2.5GB (Hugging Face models + whisper.cpp)
- Check: `docker-compose logs -f backend`
- Wait 10-15 minutes for completion

### "Database connection refused"
- Ensure postgres service is healthy: `docker-compose ps`
- Check health: `docker-compose exec postgres pg_isready -U postgres`
- Restart: `docker-compose restart postgres`

### "CORS errors in browser"
- Verify `allow_origins` in `backend/app.py` includes frontend origin
- Check nginx proxy is forwarding headers correctly

### "Transcription errors"
- Whisper.cpp model downloads automatically on first use (~466MB)
- Check logs: `docker-compose logs -f backend`
- Models stored in `/app/.cache/whispercpp_models` (Docker) or `~/.cache/whispercpp_models` (local)
- No API key needed - all processing is local

## File Locations Reference

- **API Routes**: `backend/app.py` (lines 50-180)
- **Model Loading**: `backend/app.py` (lines 41-48 startup event)
- **Whisper Transcription**: `backend/services/whisper_cpp_service.py` (local, no API)
- **Fusion Logic**: `backend/services/fusion_service.py:10-68`
- **Database Models**: `backend/models/*.py`
- **DB Init Scripts**: `db/init/*.sql` (executed in alphabetical order)
- **Frontend API Client**: `frontend/src/services/api.ts`
- **Main React App**: `frontend/src/App.tsx`
- **Docker Orchestration**: `docker-compose.yml`
- **Environment Config**: `.env` + `backend/core/config.py` (no API key required)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orosales) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
