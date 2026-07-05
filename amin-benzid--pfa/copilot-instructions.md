## pfa

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Apple Doctor** — A mobile-first web app for farmers to photograph apple leaves and receive real-time disease detection + treatment recommendations.

- **Team:** Amine Benzid (Full-Stack & Cloud), Yacine Mechri (AI Engineering)
- **Academic context:** EPI PFA 2025-2026

**Live URLs:**
- Frontend: https://d3j1y3kiqalty4.cloudfront.net
- Backend API: https://apple-doctor.duckdns.org/api
- AI Service (Hugging Face Spaces): https://aminebenzid-apple-doctor-ai.hf.space

---

## Architecture

Three independent services communicate over HTTP:

```
Frontend (React/Vite)
    → Backend (Node.js/Express) :3000
        → AI Service (FastAPI/YOLOv8) :8000
        → MongoDB Atlas
        → AWS S3 (image storage)
```

**Request flow for a diagnosis:**
1. Frontend captures image via camera and POSTs to `POST /api/predict`
2. Backend uploads image to S3 (if AWS configured), forwards to AI service
3. AI service runs YOLOv8 inference, returns detections + disease classification
4. Backend persists result to MongoDB with a UUID `request_id`, returns to client
5. Frontend renders diagnosis report with treatment recommendations

**Mock mode:** Backend has `USE_MOCK_AI=true` env var — when set, skips real AI calls and returns deterministic mock responses. This lets frontend/backend develop independently while Yacine trains the model.

**API Contract:** `docs/api-contract.md` is a **locked v1.0 spec** between backend and AI service. Breaking changes require both teams to approve. The contract defines the `/predict` request/response schema, 9 disease class taxonomy (EN/FR/AR), and error codes.

---

## Commands

### Run everything locally (Docker)
```bash
docker-compose up
```
Services: `ai-model` (:8000), `backend` (:3000), `frontend` (:5173)

### Backend
```bash
cd backend
npm install
npm run dev          # nodemon, port 3000
npm test             # Jest (--forceExit --detectOpenHandles)
```
Run a single test file:
```bash
cd backend
npx jest __tests__/predict.test.js
```

### Frontend
```bash
cd frontend
npm install
npm run dev          # Vite dev server, port 5173
npm run build        # Production build → dist/
npm run test         # Vitest (run mode, no watch)
npm run lint         # ESLint
```
Run a single test file:
```bash
cd frontend
npx vitest run src/test/treatments.test.js
```

### AI Service
```bash
cd ai-model
python -m venv venv && venv/Scripts/activate  # Windows
pip install -r requirements.txt
uvicorn src.server:app --host 0.0.0.0 --port 8000 --reload
```
Train classification model:
```bash
python src/train.py --task classify --data data/processed --epochs 10
```
Train detection model (with bboxes):
```bash
python src/train.py --task detect --data data/processed/data.yaml --epochs 50
```

### Infrastructure (Terraform)
```bash
cd infrastructure/terraform
terraform init
terraform plan
terraform apply
```

---

## Key Environment Variables

**Backend (`backend/.env`):**
| Variable | Purpose |
|---|---|
| `MONGODB_URI` | MongoDB Atlas connection string |
| `AI_SERVICE_URL` | URL of FastAPI inference service |
| `USE_MOCK_AI` | `true` skips real AI calls |
| `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | S3 uploads |
| `S3_BUCKET` | Bucket for uploaded images |
| `JWT_SECRET` | Auth token signing |

**Frontend (`frontend/.env.production`):**
| Variable | Purpose |
|---|---|
| `VITE_API_URL` | Backend API base URL in production |

**Docker Compose overrides:** `VITE_API_BASE_URL=http://localhost:3000/api`

---

## CI/CD

GitHub Actions workflows in `.github/workflows/`:

- **`ci.yml`** — Runs on every push: Jest (backend) + Vite build (frontend)
- **`deploy-backend.yml`** — After tests pass: SSH into EC2 → git pull → npm install → PM2 restart
- **`deploy-frontend.yml`** — After build: `aws s3 sync dist/` → CloudFront invalidation

Backend runs on EC2 under **PM2** process manager. EC2 is bootstrapped via `infrastructure/terraform/scripts/bootstrap.sh` (installs Node 20, PM2, writes `.env`).

---

## AWS Infrastructure

Managed entirely by Terraform in `infrastructure/terraform/`:

| File | What it provisions |
|---|---|
| `ec2.tf` | t3.micro Ubuntu instance + Elastic IP |
| `frontend.tf` | S3 bucket + CloudFront distribution (SPA 404→index.html rewrite) |
| `iam.tf` | EC2 IAM role with S3 + CloudWatch policies |
| `monitoring.tf` | CloudWatch alarms (CPU > 80%, instance down) → SNS email |
| `variables.tf` | Input vars including sensitive `mongodb_uri` |

---

## Code Notes

- **Backend** uses ES modules (`"type": "module"` in package.json); tests use Babel (`babel.config.json`) to transpile for Jest.
- **Diagnosis model** (`backend/src/models/Diagnosis.js`) stores: `request_id` (UUID, unique index), `image_url`, `detections[]`, `overall_diagnosis`, `severity`, timestamps. Indexed on `overall_diagnosis` and `createdAt` for history queries.
- **Frontend** uses `src/services/treatments.js` to map AI model class IDs to treatment recommendations — this is client-side only, not from the API.
- **History pagination:** `GET /api/history?page=1&limit=20&diagnosis=diseased` — filters and paginates from MongoDB.
- **AI model targets:** mAP@0.5 ≥ 0.75, < 500ms CPU inference, < 50MB model size.
- **9 disease classes** defined in `docs/api-contract.md`: healthy, scab, black_rot, cedar_apple_rust, powdery_mildew, fire_blight, bitter_rot, white_rot, sooty_blotch (each with EN/FR/AR labels).

---
> Source: [Amin-BenZid/PFA](https://github.com/Amin-BenZid/PFA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
