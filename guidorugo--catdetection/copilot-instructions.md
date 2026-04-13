## catdetection

> - **Stack**: FastAPI + SQLAlchemy 2.0 (async) + PostgreSQL + YOLOv8 + ResNet50 + Jinja2

# Cat Detection Project

## Quick Reference

- **Stack**: FastAPI + SQLAlchemy 2.0 (async) + PostgreSQL + YOLOv8 + ResNet50 + Jinja2
- **Python**: 3.12 (server) / 3.10 (Jetson, legacy)
- **Target hardware**: x86_64 desktop GPU server (dual GPU: RTX 2080 Ti + GTX 1070), Ubuntu 24.04
- **Dev setup**: `source venv/bin/activate` then `uvicorn app.main:app --reload` (requires a running PostgreSQL instance, or set `DATABASE_URL=sqlite+aiosqlite://` for SQLite dev mode)
- **Tests**: `pytest tests/`
- **DB migrations**: `alembic revision --autogenerate -m "description"` then `alembic upgrade head`
- **Pivot plan**: `docs/edge-saas-pivot-plan.md` — planned pivot to edge device (Radxa Cubie A7S) + cloud SaaS for litter box health monitoring. DRAFT, not yet implemented.

## Planned Pivot: Edge + SaaS ("LitterBox AI")

A pivot from the self-hosted GPU monolith to an edge+cloud architecture is planned. The current codebase is still the monolith. Key details:
- **Edge device**: Radxa Cubie A7S ($25, 3 TOPS NPU, 4GB LPDDR5) with PIR sensor + camera
- **Detection model**: Visit-based — PIR wakes system → YOLO confirms cat → record full visit → upload after cat leaves
- **Cloud**: Re-ID (CPU), health analytics, clip storage (R2), billing (Stripe), push notifications (FCM)
- **Mobile app**: Flutter (Android first)
- **~60% of current codebase reusable** in the cloud backend (models, schemas, auth, training, billing, ML)
- **Full plan**: `docs/edge-saas-pivot-plan.md`

When this pivot begins, new features should target the edge+cloud architecture. Until then, the current monolith remains the working codebase.

## Project Structure

- `app/` — Main application (FastAPI)
  - `api/` — HTTP routes and WebSocket endpoints
    - `v1/` — Versioned routes (includes `households.py`, `admin.py`, `billing.py`)
  - `core/` — Config, database, security, logging, `encryption.py`
  - `models/` — SQLAlchemy ORM models (includes `household.py`, `billing.py`, `invitation.py`, `notification_preference.py`)
  - `schemas/` — Pydantic request/response schemas
  - `services/` — Business logic (stream, detection, recording, `auto_training.py`)
  - `ml/` — ML models (YOLO detector, ResNet identifier, training)
  - `templates/` — Jinja2 HTML templates
  - `static/` — CSS/JS assets
- `scripts/` — CLI tools (setup, training, data prep, admin)
- `docker/` — Docker/compose files
- `tests/` — pytest test suite

## Key Conventions

- Always use virtual environment (`venv/`)
- Database operations are async (asyncpg for PostgreSQL, aiosqlite for SQLite dev mode)
- ML inference runs via `asyncio.to_thread()` to avoid blocking
- Frame grabbing is threaded (one thread per camera)
- Camera `source_url` must be unique (409 on duplicate)
- Recording uses FFmpeg subprocess with auto-detected codec (NVENC if available, libx264 fallback)
- YOLO class 15 = cat
- Cat Re-ID produces 512-d L2-normalized embeddings
- Stop detection pipeline during training to avoid GPU OOM

## Development Environment Rules

- **Do NOT install software on the host system.** No `sudo apt install`, `pip install` (outside the venv), `npm install -g`, or any other host-level package installation. The host is a shared production machine.
- **Always use the Python virtual environment.** Run `source venv/bin/activate` before any Python work. Install Python dependencies with `pip install` only inside the activated venv. Add new dependencies to `requirements.txt`.
- **The application runs as Docker Compose containers.** All services (catdetect, inference, training, notification, database) run via `docker/docker-compose.x86.yml`. Service-level changes must go through Dockerfiles and compose configuration, not host installs.
- **Never run `sudo` commands.** If something requires root access, escalate to the CTO/CEO.

## Multi-Tenancy

- **Household model**: users belong to exactly one household; all tenant data is scoped by `household_id`
- `get_household_context()` FastAPI dependency enforces household scoping on all tenant-aware endpoints
- Per-household file layout: `data/{household_id}/{cat_name}/` for reference images, `models/identification/{household_id}/` for trained models
- **Invite-only registration**: new users can only join via household invitation (no open signup)
- **Auto-training triggers**: the system automatically queues a training job when enough new reference images or corrections accumulate (see `app/services/auto_training.py`)

## Multi-GPU Support

- **GPU device selection**: `ML_GPU_DEVICE`, `TRAINING_GPU_DEVICE`, `ENCODING_GPU_DEVICE` settings control which GPU handles each workload
- `ENCODING_GPU_DEVICE=None` (default) falls back to `ML_GPU_DEVICE`
- System status page (`/system`) shows GPU role assignments per device
- All GPU device selection is in-process via `torch.device(f"cuda:{N}")` — no need for `CUDA_VISIBLE_DEVICES` in the main app
- Standalone servers (`training_server.py`, `inference_server.py`) use `TRAINING_GPU_DEVICE` / `INFERENCE_GPU_DEVICE` env vars mapped to `CUDA_VISIBLE_DEVICES`
- CPU-only mode: works without CUDA for CI and non-GPU deployments

## Camera Protocols

- `source_type` field on cameras: `"rtsp"`, `"mjpeg"`, `"http"`, `"v4l2"`, `"auto"` (auto-inferred from URL)
- RTSP: uses `cv2.CAP_FFMPEG` with configurable transport (`CAMERA_RTSP_TRANSPORT`, default `tcp`)
- MJPEG/HTTP (ESP32-CAM): uses `cv2.CAP_FFMPEG` with configurable timeout (`CAMERA_MJPEG_TIMEOUT`, default 10s)
- V4L2 (USB cameras): uses `cv2.CAP_V4L2` with MJPG FOURCC (YUYV is USB-bandwidth-limited at higher resolutions)
- Per-camera `resolution` field (e.g. `"1280x720"`, `NULL` = camera default) — sets `CAP_PROP_FRAME_WIDTH/HEIGHT` on V4L2
- Per-camera `expected_fps` controls `CAP_PROP_FPS` on V4L2 — selectable in the UI when adding/editing cameras
- V4L2 uses default buffer count (4); single-buffer mode (`BUFFERSIZE=1`) drops ~50% of frames due to kernel blocking
- Protocol-specific reconnect backoff (shorter for MJPEG/ESP32, standard for RTSP)
- Schema validators enforce URL format matches source_type when explicit

## Swann NVR/DVR Integration

- Swann NVRs expose standard RTSP streams — no backend pipeline changes needed, cameras are added as `source_type: "rtsp"`
- "Add NVR Cameras" wizard on `/cameras` page auto-generates RTSP URLs from NVR connection details + channel selection
- `GET /api/v1/cameras/nvr-presets` returns URL format presets for Swann OEM variants (Hikvision, Raysharp, generic)
- `POST /api/v1/cameras/test-connection` validates any camera URL by opening a transient `cv2.VideoCapture` and reading one frame
- Four presets: Raysharp-based DVR (default), Hikvision-based NVR, generic preview, single-stream legacy
- URL generation is client-side; passwords are percent-encoded via `encodeURIComponent()`
- Wizard is 3-step: Connection (test ch1) → Channel selection (per-channel test) → Names & save
- Some DVRs crash with 3+ mainstream RTSP connections — wizard warns and defaults to substreams
- No new env vars, no database changes, no Alembic migration

## Hardware Encoding (HEVC/H.264)

- `RECORDING_CODEC` env var: `"auto"` (default, HEVC preferred), `"hevc_auto"`, `"h264_auto"`, `"hevc_nvenc"`, `"libx265"`, `"h264_nvenc"`, `"libx264"`
- **Runtime config**: `config/recording_config.json` overrides env var — editable via System page UI toggle
- **Resolution priority**: runtime JSON config > env var `RECORDING_CODEC`
- Auto mode probes `ffmpeg -encoders` for both `hevc_nvenc` and `h264_nvenc`
- `auto` preference order: `hevc_nvenc` > `h264_nvenc` > `libx265`
- NVENC uses `-gpu {ENCODING_GPU_DEVICE}` to target a specific GPU
- `RECORDING_NVENC_PRESET`: NVENC preset (`p1`=fastest to `p7`=slowest, default `p4`)
- HEVC recordings require `-tag:v hvc1` for browser/Apple compatibility (handled automatically)
- Recording.codec DB field stores the actual codec used (`"hevc_nvenc"`, `"libx265"`, `"h264_nvenc"`, `"libx264"`)
- Fallback chain: `hevc_nvenc` → `libx265` → `libx264`; `h264_nvenc` → `libx264`
- `GET /api/v1/recordings/{id}/stream-h264` — on-demand HEVC→H.264 transcode for unsupported browsers
- Frontend auto-falls back to `/stream-h264` if `<video>` playback fails (transparent to user)
- Notification system is unaffected — GIFs are generated independently from buffer frames

## Docker

### x86_64 (primary)
- **Base image**: `nvidia/cuda:12.6.0-cudnn-runtime-ubuntu24.04` (forward-compatible with CUDA 13.0 driver)
- `docker/Dockerfile.app` — Full application with FFmpeg + NVENC support
- `docker/Dockerfile.training` — Standalone training server
- `docker/Dockerfile.inference` — Standalone inference server
- `docker/Dockerfile.notification` — Lightweight notification server (`python:3.12-slim`, no GPU)
- `docker/docker-compose.x86.yml` — Unified microservices stack (catdetect + inference + training + notification), optional `testing` profile for dummy camera
- `docker/docker-compose.x86.inference.yml` — Standalone inference
- `docker/docker-compose.x86.training.yml` — Standalone training
- CPU-only: remove `deploy.resources.reservations.devices`, set `RECORDING_CODEC=libx264`
- ROCm (AMD): see comments in compose files for alternative base image and device config

### Microservices Architecture (v3.0.0)
- `docker-compose.x86.yml` runs all services in a unified stack:
  - `db` — PostgreSQL database (`postgres:16-alpine`), internal only
  - `catdetect` — Main app (YOLO + web UI + API), port 8100:8000
  - `inference` — Re-ID embedding server, internal only
  - `training` — Training server, internal only
  - `notification` — Telegram/Slack/webhook delivery, internal only
  - `dummy-stream` — Test camera (testing profile), port 8090
- Services communicate via Docker internal networking (e.g., `http://inference:8002`)
- Only the main app exposes an external port
- Shared volumes: `models/`, `data/`, `recordings/`, `thumbnails/`
- `notification-config` named volume persists notification channel configuration
- Process names visible in `nvidia-smi`/`ps`: `catdetect-app`, `catdetect-infer`, `catdetect-train`, `catdetect-notif`

### Jetson (legacy, kept for reference)
- `docker/Dockerfile.jetson` — Jetson L4T image (NOT actively maintained)
- `docker/docker-compose.yml` — Jetson compose (NOT actively maintained)
- Build requires `network: host` in docker-compose build section (Jetson iptables issue)
- `numpy` must be pinned to `<2` — base image PyTorch is compiled against numpy 1.x
- `bcrypt` pinned to `<5.0.0` for passlib compatibility
- ML imports in `app/main.py` are lazy (inside lifespan) to allow API-only mode when torch is unavailable

## Remote Training (Client-Server)

- Server runs `scripts/training_server.py` (standalone FastAPI daemon, no `app.*` imports)
- Jetson runs `scripts/training_client.sh` to push data, trigger training, poll, and pull the model
- **Data upload via HTTP**: `POST /upload-data` on the training server accepts a tar archive (`application/octet-stream`). This replaces the old rsync-over-SSH approach — no SSH key setup required
- **UI remote training**: select "Train on remote server" in the training page (requires `TRAINING_SERVER_URL` or `TRAINING_SERVER_SSH` + `TRAINING_API_KEY` in `.env`). The app uploads data via HTTP, triggers training, polls progress, downloads the model, and optionally hot-reloads — all from the browser
- **Auto-activate toggle**: training and hyperparameter search forms have an "Auto-activate" checkbox (default on). When unchecked, the model is registered but not activated — user can manually activate from the Models section after reviewing metrics
- `TRAINING_SERVER_URL` (e.g. `http://10.0.0.82:8001`) takes priority over SSH-derived URL. If not set, URL is derived from `TRAINING_SERVER_SSH` (backward compatible). This makes `TRAINING_SERVER_SSH` optional for HTTP-only setups
- Server handles both `prepare_data.py` (YOLO cropping) and `train_identifier.py` as subprocesses
- Auth via `X-API-Key` header (shared `TRAINING_API_KEY` env var)
- PyTorch `.pth` files are portable between x86_64 ↔ ARM64 (`map_location=device`)
- TensorRT `.engine` files are NOT portable (device-specific), but the app falls back to `.pth`
- `POST /api/v1/training/reload-model` hot-swaps the identifier model without restarting the app
- `POST /api/v1/training/jobs/{id}/cancel` cancels a running/pending training job
- Training UI shows a live progress bar with auto-polling every 5s
- On startup, orphaned remote training jobs (stuck as "running" after a restart) are automatically resumed — polling continues, model is downloaded and hot-reloaded on completion. Orphaned local jobs are marked as failed.
- SQLite stays on Jetson only; the server never touches the database
- `scripts/sync_and_train.sh` is deprecated (kept for reference)

## Modal Cloud Training (v3.1.0)

- **Third `training_location` option**: `"modal"` alongside `"local"` and `"remote"`
- Serverless GPU training via [Modal](https://modal.com) — per-second billing, zero idle costs
- Modal app definition: `modal_app/train_remote.py` (deploy with `modal deploy modal_app/train_remote.py`)
- Training wrapper: `modal_app/training_wrapper.py` (standalone trainer, no FastAPI/DB deps)
- Client service: `app/services/modal_training.py` (`ModalTrainingClient` singleton)
- **Data flow**: tar household images → upload to Modal Volume → spawn GPU function → poll progress via Volume JSON → download `.pth` model → register locally
- **No pipeline pause** — Modal runs on cloud GPU, detection keeps running locally
- **Progress tracking**: training function writes `/vol/progress/{household_id}.json` to Volume every epoch; client polls it every 10s
- **Orphan recovery**: `modal_call_id` saved in TrainingJob config JSON; resumed via `FunctionCall.from_id()` on restart
- **Cancel**: `FunctionCall.from_id(call_id).cancel()` — best-effort
- **Auto-training**: `AUTO_TRAINING_LOCATION` env var controls where auto-triggered jobs run (default: `"local"`)
- **GPU types**: `MODAL_GPU_TYPE` env var — T4 ($0.59/hr), L4 ($0.80/hr), A10G ($1.10/hr), A100 ($2.10/hr)
- **Free tier**: $30/mo credits on Modal Starter plan — covers ~50 training runs on A10G
- **Setup**: `modal token new` → set `MODAL_TOKEN_ID` + `MODAL_TOKEN_SECRET` in `.env` → `modal deploy modal_app/train_remote.py`
- UI: "Modal (Cloud)" radio button in training page with GPU type selector; disabled if not configured

## Training Job Queue

- Multiple training jobs can be submitted; if one is running, subsequent jobs are queued (`status: "queued"`)
- Queued jobs run sequentially in FIFO order (by `created_at`)
- Detection pipeline stays paused for the entire queue drain (not pause/resume between each job)
- `_run_training_wrapper` orchestrates: run current job → dequeue next → repeat until queue empty
- `_dequeue_next_job` picks the oldest queued standalone job and transitions it to `"pending"`
- Hyperparameter searches cannot be queued — they require all queued jobs to be cancelled first
- Queued jobs survive app restart: `_process_queue_on_startup` runs after orphan cleanup
- Resumed remote orphan jobs trigger queue processing in their `finally` block
- Cancel a queued job without affecting running or other queued jobs
- No database migration required — `status` is a `String(20)` column, not an enum

## Test Page

- `POST /api/v1/test/detect` — upload a photo, detect cats (YOLO) + identify each (ResNet Re-ID + embedding matching), returns JSON with bounding boxes, confidence, cat names, and similarity scores
- `POST /api/v1/test/feedback` — submit identity correction: crops image at bbox, saves crop to `data/{cat_name}/` as reference image, generates embedding, and rebuilds in-memory embedding store for that cat
- Handles multiple cats per image; each detection is independently identified
- UI at `/test` draws bounding boxes on a canvas with color-coded labels
- "Draw Box" mode allows manually drawing bounding boxes for missed detections (click+drag on canvas)
- Each detection (YOLO or manual) shows a dropdown to correct the cat identity + Save button
- Feedback crops are saved as `feedback_{timestamp}.jpg` and appear in the cat's image gallery
- **Important**: Feedback improves *identification* (Re-ID) only — it adds reference images/embeddings so the system better recognizes *which* cat it sees. It does NOT retrain the YOLO detector; missed detections require lowering the confidence threshold, using a larger YOLO model, or fine-tuning YOLO on custom data

## Model Management

- `GET /api/v1/models` — list all registered model versions with metrics and active status
- `POST /api/v1/models/{version}/activate` — activate a model version and hot-reload it into the pipeline (rebuilds embedding store)
- Training page includes a "Models" section showing all versions with an "Activate" button
- **Model versioning**: format is `YYYYMMDD_j{job_id}` (e.g., `20260402_j42`) — deterministic, links to DB training job ID. Standalone scripts use `YYYYMMDD_t{unix_ts}` as fallback.
- **Model registry**: per-household JSON file at `models/identification/{household_id}/registry.json`. The global registry at `models/identification/registry.json` is deprecated — all new code must use `ModelRegistry(household_id=...)`.
- **Startup model loading**: scans household registries first, falls back to global registry with a deprecation warning.
- **Embedding validation**: on startup, embeddings from DB are validated against the active model version. Mismatched embeddings are skipped with a warning (regenerate with `POST /api/v1/cats/generate-embeddings`).

## Cat Image Management

- Reference images stored in `data/{cat_name}/` (case-insensitive directory lookup)
- `GET /api/v1/cats/{id}/images` — list reference images (jpg/jpeg/png)
- `POST /api/v1/cats/{id}/images` — upload one or more reference images (multipart, multiple files)
- `GET /api/v1/cats/{id}/images/{filename}` — serve individual image
- `DELETE /api/v1/cats/{id}/images/{filename}` — delete individual image
- `POST /api/v1/cats/generate-embeddings` — generate reference embeddings from training data images (auto-triggered after training)
- Cats page shows thumbnail grid per cat with upload (+) button, lightbox preview, and per-image delete
- `_find_cat_dir()` helper handles case-insensitive directory matching (DB names are capitalized, data dirs are lowercase)

## Training Hyperparameters

- **Recommended LR**: `0.0001` (1e-4) — consistently produces 95%+ Rank-1 accuracy
- **Avoid**: `0.00001` (1e-5) — too low for triplet loss + cosine annealing, plateaus at ~83-88% even with 2x epochs
- Cosine annealing scheduler (`CosineAnnealingLR(T_max=epochs)`) decays LR to near-zero mid-cycle; base LR must be high enough that the minimum is still usable
- `REID_IMAGE_SIZE` (default 512) controls input resolution for training and inference; stored in checkpoint as `img_size`
- `TripletBatchSampler` samples P cats × K=8 images per batch; cats with fewer images are oversampled with replacement

## Hyperparameter Search

- `POST /api/v1/training/search` — start a search over all combinations of learning rates, epochs, and freeze epochs
- `GET /api/v1/training/searches` — list searches (paginated `{items, total, limit, offset}`)
- `GET /api/v1/training/searches/{id}` — get search with all trial details
- `POST /api/v1/training/searches/{id}/cancel` — cancel search + running trial
- Trials run sequentially; detection pipeline is paused once for the entire search
- Data preparation runs only on the first trial; subsequent trials reuse processed data
- On completion, the best model (by rank-1 accuracy) is automatically registered, activated, and embeddings are regenerated
- `HyperparamSearch` model links to `TrainingJob` via `search_id` FK and `trial_number`
- `_run_training_job()` and `_run_remote_training_job()` accept `skip_post_training=True` to skip model registration/reload/embedding generation (caller handles it)
- Orphaned searches are handled on startup: remote searches resume, local searches are marked failed
- Training UI shows a "Hyperparameter Search" button with comma-separated parameter inputs and live combination counter

## 30-Second Detection Window

- When a cat is first detected on a camera, a 30-second detection window opens
- All frames within the window are processed: YOLO detect → crop → Re-ID embedding → majority vote per tracked cat
- Cats are tracked across frames by bbox centroid proximity (`DETECTION_WINDOW_MAX_CENTROID_DIST`, default 150px)
- When the window expires, one MP4 clip is encoded from the rolling buffer and events are created with majority-vote identity
- Tracks with fewer than `DETECTION_WINDOW_MIN_VOTES` (default 3) frames are discarded (noise filter)
- Multiple cats in the same window share one recording, each gets a separate event
- If a cat is still visible when the window expires, a new consecutive window opens immediately
- Annotated frames (with bboxes baked in) are stored in the buffer, so recordings include visual overlays
- `DetectionWindowManager` in `app/services/detection_window.py`, integrated in `DetectionPipeline._process_detections()`
- `RecordingService.encode_clip()` encodes buffered JPEG frames to MP4 via FFmpeg (replaces the old `trigger_recording()` live-capture flow)
- Rolling buffer holds `DETECTION_WINDOW_BUFFER_SECONDS` (default 35s) of JPEG frames per camera (~17.5 MB/camera at 5fps)

## Event Identity Correction

- `POST /api/v1/events/{event_id}/feedback` — correct a detection's cat identity (body: `{cat_id: int}`)
- Reads the event thumbnail, saves it as a reference image in `data/{cat_name}/`, generates embedding, updates the event with `is_corrected=True`
- Events UI shows "Correct Identity" button per event group with a cat selection modal
- `GET /api/v1/events/stats` includes `corrections_since_training` and `suggest_retrain` (true when corrections >= `RETRAIN_SUGGESTION_THRESHOLD`)
- Events page shows a retrain suggestion banner when threshold is reached

## Event Validation & False Positive Dismissal

- `POST /api/v1/events/{event_id}/validate` — confirm a detection's identity is correct, save crop as reference image and generate embedding
- `POST /api/v1/events/{event_id}/dismiss` — mark a detection as false positive (`is_false_positive = True`)
- `POST /api/v1/events/{event_id}/restore` — un-dismiss a previously dismissed detection
- Dismissed events are excluded from stats (total count, today count, per-cat counts)
- VALIDATE button in the UI skips both already-validated and dismissed events
- Expanded detection groups show per-detection Dismiss/Undo buttons
- Dismissed events appear muted with a "dismissed" badge
- Cannot validate a dismissed event (returns 400)

## Retention Cleanup

- `CleanupService` runs hourly, deletes events + thumbnails + recordings older than `EVENT_RETENTION_DAYS` (default 7)
- Started in app lifespan, stopped on shutdown

## Video-Centric Events UI

- Events page groups detections by recording — shows inline `<video>` player when a recording exists
- `GET /api/v1/recordings/{id}/stream` accepts JWT via `token` query param (for `<video src>` compatibility), supports HTTP Range requests for seeking
- Events without recordings fall back to thumbnail display
- Large play button overlay on video thumbnails for easy playback
- "Group by Recording" toggle shows all cats detected in each recording with per-cat correction buttons
- Sibling cat awareness: when grouped by cat, shows "Also in recording: Milo" for multi-cat recordings
- Per-event "Correct" button in expanded detection details for fine-grained identity correction

## API Pagination

- `GET /api/v1/events` and `GET /api/v1/training/jobs` return paginated responses: `{items, total, limit, offset}`
- Events default 50 per page, training jobs default 10 per page

## Remote Inference (Re-ID Offloading)

- `scripts/inference_server.py` — standalone FastAPI daemon on the GPU server (no `app.*` imports, same auth pattern as training server)
- Jetson sends JPEG crops to the server, server returns 512-d embeddings; Jetson keeps embedding store and does matching locally
- `GET /health` — server status + model loaded check
- `POST /embed` — single JPEG body → 512-d embedding
- `POST /embed-batch` — multipart N JPEGs → N embeddings (single batch forward pass)
- `POST /reload` — reload model from disk (called automatically after training)
- `RemoteIdentifier` in `app/ml/identifier.py` wraps remote calls with circuit breaker (3 failures → fallback to local → health check every 30s → auto-recover)
- `docker/Dockerfile.inference` — lightweight CUDA image with only torch/torchvision/fastapi
- `docker/docker-compose.server.yml` — server-side compose with `training` + `inference` services sharing `models/` volume
- After training, `_reload_model_after_training()` automatically calls `POST /reload` on the inference server
- `INFERENCE_FALLBACK_LOCAL=false` skips loading ResNet50 locally (saves ~100MB GPU on Jetson)

## External Notifications

- `scripts/notification_server.py` — standalone FastAPI daemon (no `app.*` imports, same auth pattern as inference/training)
- Delivers detection alerts to Telegram, Slack, and webhook endpoints
- One notification per 30-second detection window (not per individual event)
- Config stored in JSON file (`/app/config/notification_config.json`, Docker named volume)
- Per-camera, per-channel cooldown prevents spam during continuous activity
- Each channel can override the global default cooldown (60s); `cooldown_seconds: null` on a channel means use global default
- Retry: 3 attempts with exponential backoff (1s, 2s, 4s) via `httpx.AsyncClient`
- `docker/Dockerfile.notification` — lightweight `python:3.12-slim` image (~150MB, no GPU/torch/OpenCV)
- **Telegram**: sends animated GIF via `sendAnimation` (falls back to photo, then text)
- **Slack**: text message via incoming webhook
- **Webhook**: raw JSON payload to configurable URL (includes `animation_base64` GIF when available)
- **Notification GIF**: animated clip generated from recording buffer frames at window finalization
  - Subsampled to `NOTIFICATION_GIF_MAX_FRAMES` (default 20), resized to `NOTIFICATION_GIF_WIDTH` (default 480px)
  - Encoded with PIL as animated GIF, runs in a thread to avoid blocking
  - `_encode_notification_gif()` in `app/services/detection_pipeline.py`
- `ExternalNotificationClient` in `app/services/external_notifications.py` wraps HTTP calls to notification server
- Injected into `DetectionPipeline` — calls `notify_detection_window()` after `_finalize_window()` creates events
- Proxy API at `/api/v1/notifications/*` centralizes JWT auth (all requests forwarded with `X-API-Key`)
- UI at `/notifications` — global enable/disable, default cooldown, per-channel cooldown, channel CRUD, test buttons
- `NOTIFICATION_SERVER_URL`, `NOTIFICATION_API_KEY`, `NOTIFICATION_SERVER_TIMEOUT` env vars

## Batched Detection

- `CatDetector.detect_batch(frames)` processes all camera frames in one YOLO inference call
- `DetectionPipeline._run()` collects frames from all cameras, runs batched YOLO, then processes per-camera detections
- Stale-frame skip: detection loop tracks `frame_time` per camera and skips frames already processed (avoids wasted YOLO inference when camera FPS < `DETECTION_FPS`)
- `CatIdentifier.get_embeddings_batch(crops)` and `RemoteIdentifier.get_embeddings_batch(crops)` batch all crops from a camera
- `export_tensorrt(batch_size=N)` exports TensorRT engine with multi-batch support
- Thumbnail JPEG encoding offloaded to `asyncio.to_thread()` (was sync in event loop)

## Timestamp Overlay

- `_draw_timestamp(frame)` draws wall-clock time on frames in-place (semi-transparent black background, white text)
- Applied **after** bbox annotation but **before** recording buffer and live feed storage
- Thumbnails (encoded inside the detection loop) do NOT get the timestamp — crops stay clean
- Raw broadcast frames (between detection cycles) get timestamp drawn on a copy
- Controlled by `TIMESTAMP_OVERLAY` (on/off), `TIMESTAMP_FORMAT` (strftime), `TIMESTAMP_POSITION` (bottom-left/top-left)

## Live Feed Broadcasting

- Broadcast loop runs at `LIVE_FEED_FPS` (default 15), decoupled from `DETECTION_FPS` (default 5)
- Prefers annotated frames (with bboxes + timestamp) from the detection pipeline; falls back to raw grabber frames between detection cycles for smoother video
- `_annotated_frames` stores `(frame, frame_time)` tuples — updated after bbox annotation + timestamp (not before), so live feed always shows detection overlays
- Deduplicates on `frame_time` to avoid broadcasting the same frame twice
- JPEG encoding uses GPU batch (`torchvision.io.encode_jpeg`) when `LIVE_FEED_GPU_JPEG=true` and CUDA is available; falls back to CPU-threaded `cv2.imencode`
- All JPEG encoding runs via `asyncio.to_thread()` to avoid blocking the async event loop
- `RecordingBuffer.add_frame()` also offloaded to thread (CPU JPEG encode ~3-5ms)

## System Status Page

- `/system` — UI page showing live system health and configuration
- `GET /api/v1/system/status` — full system status (app info, GPU, pipeline, ML models, embeddings, cameras, remote services config)
- `GET /api/v1/system/inference-health` — proxy health check to remote inference server
- `GET /api/v1/system/training-health` — proxy health check to remote training server
- `GET /api/v1/system/notification-health` — proxy health check to notification server
- Cards: Application, GPU, Detection Pipeline, ML Models, Cameras, Inference Server, Training Server, Notification Server
- Status badges show real-time state (connected/degraded/disconnected/offline)
- Camera health uses hysteresis: non-critical transitions (connected↔degraded) require 3 consecutive checks (~6s) to prevent flapping; critical transitions (disconnected/offline) are immediate
- Refresh button re-fetches all status data; remote server health checked in parallel

## Low-Light Detection

- Automatic brightness-based adaptation — no manual mode switching required
- `compute_brightness()` measures mean luminance of each frame (~0.2ms for 720p)
- Three light levels: `"normal"` (>=80), `"low_light"` (40-79), `"very_dark"` (<40) — thresholds configurable
- Per-camera EMA smoothing (alpha=0.3) prevents frame-to-frame jitter
- Hysteresis (3 consecutive frames) prevents flapping between levels — mirrors `CameraHealthMonitor` pattern
- **Adaptive YOLO confidence**: raised in low light to reduce false detections (post-filter per camera)
- **Adaptive Re-ID threshold**: raised in low light to prevent wrong identifications — prefers "unknown" over wrong name
- **CLAHE crop enhancement**: applies Contrast Limited Adaptive Histogram Equalization to crops before Re-ID in low light (LAB L-channel, preserves color)
- **Notification suppression**: suppresses external notifications in `very_dark` unless detection confidence exceeds threshold
- `brightness` and `light_level` stored on `DetectionEvent` for analytics
- Per-camera brightness broadcast via WebSocket (`camera_brightness` status message)
- `/api/v1/system/status` includes `low_light` section with per-camera brightness and light level
- All logic in `app/services/brightness.py` (`BrightnessTracker`, `compute_brightness`, `classify_light_level`, `get_adaptive_thresholds`, `enhance_crop_for_reid`)

## Dashboard Theater Mode

- Click any camera feed on the dashboard to open fullscreen theater mode
- Theater overlay shows camera name, status badge, and live FPS counter
- Frames are mirrored from the grid canvas WebSocket — no additional connection
- Close with Escape key or close button
- CSS in `app/static/css/style.css` (`.theater-*` classes), JS in `app/static/js/stream.js`

## Dummy Camera Stream

- `scripts/dummy_stream.py` — Standalone MJPEG-over-HTTP server simulating an ESP32-CAM
- `GET /stream` — MJPEG multipart boundary stream
- `GET /snapshot` — Single JPEG frame
- `GET /` — Simple HTML page showing the stream
- Generates synthetic frames: colored background, timestamp, moving shape
- Video file mode: reads frames from an MP4/AVI and loops
- Usage: `python scripts/dummy_stream.py --port 8090 --fps 5`
- Available as Docker Compose `testing` profile in `docker-compose.x86.yml`

## CI

- `.github/workflows/test.yml` — GitHub Actions: runs pytest on push/PR to main
- Uses CPU-only PyTorch (`--index-url https://download.pytorch.org/whl/cpu`)
- All tests mock GPU/camera/FFmpeg — no GPU needed in CI


## Environment Variables

See `.env.example` for all available settings.

| Setting | Default | Description |
|---------|---------|-------------|
| `DATABASE_URL` | postgresql+asyncpg://... | Database connection string (use `sqlite+aiosqlite://` for dev) |
| `DB_PASSWORD` | None | PostgreSQL password (used in default DATABASE_URL construction) |
| `ML_GPU_DEVICE` | 0 | GPU index for YOLO + Re-ID inference |
| `TRAINING_GPU_DEVICE` | 0 | GPU index for training jobs |
| `TRAINING_CPU_WORKERS` | None | CPU threads for training/data-prep (None = cpu_count - 1) |
| `ENCODING_GPU_DEVICE` | None | GPU index for NVENC encoding (None = same as ML_GPU_DEVICE) |
| `RECORDING_CODEC` | auto | Encoding codec: `"auto"` (HEVC preferred), `"hevc_auto"`, `"h264_auto"`, `"hevc_nvenc"`, `"libx265"`, `"h264_nvenc"`, `"libx264"` |
| `RECORDING_NVENC_PRESET` | p4 | NVENC preset (p1=fastest, p7=slowest) |
| `LIVE_FEED_FPS` | 15 | Target FPS for live dashboard feed (decoupled from detection) |
| `CAMERA_RTSP_TRANSPORT` | tcp | RTSP transport: `"tcp"` or `"udp"` |
| `CAMERA_MJPEG_TIMEOUT` | 10.0 | MJPEG/HTTP connection timeout in seconds |
| `DETECTION_WINDOW_SECONDS` | 30 | Duration of each detection window |
| `DETECTION_WINDOW_BUFFER_SECONDS` | 35 | Rolling buffer size (5s margin over window) |
| `DETECTION_WINDOW_MAX_CENTROID_DIST` | 150.0 | Max pixel distance for cross-frame bbox association |
| `DETECTION_WINDOW_MIN_VOTES` | 3 | Min frames a track needs to create an event |
| `EVENT_RETENTION_DAYS` | 7 | Auto-delete events/recordings older than this |
| `REID_IMAGE_SIZE` | 512 | Re-ID model input size |
| `RETRAIN_SUGGESTION_THRESHOLD` | 15 | Suggest retrain after N identity corrections |
| `TRAINING_SERVER_URL` | None | Remote training server URL (e.g. `http://10.0.0.82:8001`), overrides SSH-derived URL |
| `INFERENCE_SERVER_URL` | None | Remote inference server URL (e.g. `http://10.0.0.82:8002`) |
| `INFERENCE_SERVER_API_KEY` | None | X-API-Key for inference server |
| `INFERENCE_SERVER_TIMEOUT` | 5.0 | HTTP timeout in seconds for inference requests |
| `INFERENCE_FALLBACK_LOCAL` | true | Fall back to local ResNet50 if server is down |
| `NOTIFICATION_SERVER_URL` | None | Notification server URL (e.g. `http://notification:8003`) |
| `NOTIFICATION_API_KEY` | None | X-API-Key for notification server |
| `NOTIFICATION_SERVER_TIMEOUT` | 5.0 | HTTP timeout in seconds for notification requests |
| `TIMESTAMP_OVERLAY` | true | Draw timestamp on live feed and recordings |
| `TIMESTAMP_FORMAT` | %Y-%m-%d %H:%M:%S | strftime format for timestamp overlay |
| `TIMESTAMP_POSITION` | bottom-left | Timestamp position: `"bottom-left"` or `"top-left"` |
| `LIVE_FEED_GPU_JPEG` | true | Use GPU (torchvision) for live feed JPEG encoding |
| `LOW_LIGHT_ENABLED` | true | Enable automatic low-light detection adjustments |
| `LOW_LIGHT_THRESHOLD` | 80.0 | Mean luminance below this = low_light |
| `VERY_DARK_THRESHOLD` | 40.0 | Mean luminance below this = very_dark |
| `LOW_LIGHT_CONFIDENCE_BOOST` | 0.15 | Added to DETECTION_CONFIDENCE in very_dark |
| `LOW_LIGHT_REID_BOOST` | 0.15 | Added to IDENTIFICATION_THRESHOLD in very_dark |
| `LOW_LIGHT_ENHANCE_CROPS` | true | Apply CLAHE to crops before Re-ID in low-light |
| `LOW_LIGHT_SUPPRESS_NOTIFICATIONS` | true | Suppress notifications in very_dark unless high confidence |
| `LOW_LIGHT_NOTIFICATION_MIN_CONFIDENCE` | 0.8 | Min detection confidence to notify in very_dark |
| `NOTIFICATION_GIF_ENABLED` | true | Send animated GIF instead of static image in notifications |
| `NOTIFICATION_GIF_MAX_FRAMES` | 20 | Max frames in the notification GIF |
| `NOTIFICATION_GIF_WIDTH` | 480 | Resize width for GIF frames (0 = no resize) |
| `NOTIFICATION_GIF_FPS` | 4 | GIF playback speed in frames per second |
| `TELEGRAM_BOT_TOKEN` | None | Telegram bot token for direct notification delivery |
| `STRIPE_SECRET_KEY` | None | Stripe secret API key for billing |
| `STRIPE_PUBLISHABLE_KEY` | None | Stripe publishable key (sent to frontend) |
| `STRIPE_WEBHOOK_SECRET` | None | Stripe webhook signing secret |
| `BILLING_ENABLED` | false | Enable Stripe billing integration |
| `ENCRYPTION_AT_REST_ENABLED` | false | Enable field-level encryption at rest |
| `ENCRYPTION_KEY` | None | Fernet key for encryption at rest (required if encryption enabled) |
| `MODAL_TOKEN_ID` | None | Modal API token ID (from `modal token new`) |
| `MODAL_TOKEN_SECRET` | None | Modal API token secret |
| `MODAL_GPU_TYPE` | A10G | Modal GPU type: T4, L4, A10G, A100, H100 |
| `MODAL_APP_NAME` | cat-reid-training | Deployed Modal app name |
| `MODAL_VOLUME_NAME` | cat-reid-data | Modal Volume name for data/models |
| `AUTO_TRAINING_LOCATION` | local | Default training location for auto-triggered jobs: "local", "remote", or "modal" |

## Households API

- `POST /api/v1/households` — create a new household (creates the household and assigns the current user as owner)
- `GET /api/v1/households/current` — get the current user's household details
- `PUT /api/v1/households/current` — update household settings
- `POST /api/v1/households/current/invitations` — create an invitation link for new members
- `GET /api/v1/households/current/members` — list household members
- `DELETE /api/v1/households/current/members/{user_id}` — remove a member from the household
- All household-scoped endpoints use `get_household_context()` dependency

## Admin API

- `GET /api/v1/admin/households` — list all households (admin only)
- `GET /api/v1/admin/users` — list all users (admin only)
- `POST /api/v1/admin/users/{user_id}/role` — change a user's role (admin only)
- `GET /api/v1/admin/stats` — system-wide statistics (household count, user count, event count)

## Billing API

- `POST /api/v1/billing/checkout` — create a Stripe checkout session for a subscription plan
- `GET /api/v1/billing/subscription` — get current household subscription status
- `POST /api/v1/billing/portal` — create a Stripe customer portal session for managing subscriptions
- `POST /api/v1/billing/webhook` — Stripe webhook handler (verifies signature via `STRIPE_WEBHOOK_SECRET`)
- Only active when `BILLING_ENABLED=true`

## Onboarding Flow

- New users register via invitation link (invite token required)
- On first login, user is guided through household setup: name household, add first camera, upload initial cat photos
- Auto-training triggers after initial reference images are uploaded

## Post-Reboot: Training Jobs to Queue

After reboot, ensure GPU fan control service is running (`sudo systemctl status gpu-fan-control.service`), then generate a fresh token and queue these 9 jobs.

### Generate API token
```bash
cd /multimedia/projects/CatDetection
./venv/bin/python3 -c "
from app.core.security import create_access_token
token = create_access_token('admin')
print(token)
"
```

Set `TOKEN=<output>` then run the curl commands below.

### Batch 1: Re-run top 3 (same params as best historical jobs)

```bash
# Job 4 clone: Best ever Rank-1=0.9599 — lr=0.0001, epochs=300, freeze=5
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":300,"learning_rate":0.0001,"batch_size":24,"freeze_epochs":5,"prepare_data":true,"auto_activate":true,"training_location":"local","household_id":1}'

# Job 6 clone: Rank-1=0.9531 — lr=0.0001, epochs=300, freeze=5
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":300,"learning_rate":0.0001,"batch_size":24,"freeze_epochs":5,"prepare_data":false,"auto_activate":true,"training_location":"local","household_id":1}'

# Job 3 clone: Rank-1=0.9270 — lr=0.0001, epochs=250, freeze=5
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":250,"learning_rate":0.0001,"batch_size":24,"freeze_epochs":5,"prepare_data":false,"auto_activate":true,"training_location":"local","household_id":1}'
```

### Batch 2: Scaled-up versions of top 3 (more epochs + longer warmup)

```bash
# Job 4 scaled: 500 epochs, freeze=10 (longer warmup for deeper convergence)
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":500,"learning_rate":0.0001,"batch_size":24,"freeze_epochs":10,"prepare_data":false,"auto_activate":true,"training_location":"local","household_id":1}'

# Job 6 scaled: 500 epochs, freeze=15 (even more backbone warmup)
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":500,"learning_rate":0.0001,"batch_size":24,"freeze_epochs":15,"prepare_data":false,"auto_activate":true,"training_location":"local","household_id":1}'

# Job 3 scaled: 400 epochs, freeze=10
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":400,"learning_rate":0.0001,"batch_size":24,"freeze_epochs":10,"prepare_data":false,"auto_activate":true,"training_location":"local","household_id":1}'
```

### Batch 3: Experimental optimized configs

Rationale: The top jobs all use lr=0.0001. The cosine annealing scheduler decays LR to near-zero over `epochs` steps. With 300 epochs the effective learning window may be too narrow. These configs explore: (a) slightly higher LR with longer freeze to let the backbone stabilize before fine-tuning, (b) very long training at the sweet-spot LR, (c) a two-phase approach with higher freeze to protect pretrained features longer.

```bash
# Experiment A: lr=0.00015, 400 epochs, freeze=15
# Slightly higher LR gives cosine annealing more headroom; longer freeze
# protects pretrained ResNet50 features during high-LR early epochs
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":400,"learning_rate":0.00015,"batch_size":24,"freeze_epochs":15,"prepare_data":false,"auto_activate":true,"training_location":"local","household_id":1}'

# Experiment B: lr=0.0001, 600 epochs, freeze=20
# The sweet-spot LR with maximum training time; freeze=20 ensures the backbone
# is thoroughly warmed up before end-to-end fine-tuning. Cosine annealing over
# 600 epochs gives a gentler decay curve than 300.
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":600,"learning_rate":0.0001,"batch_size":24,"freeze_epochs":20,"prepare_data":false,"auto_activate":true,"training_location":"local","household_id":1}'

# Experiment C: lr=0.00008, 500 epochs, freeze=10
# Slightly lower than the sweet spot but higher than 1e-5 (which plateaus).
# Tests whether a marginally lower peak LR with more epochs can beat 0.0001/300.
curl -s -X POST http://localhost:8100/api/v1/training/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"model_type":"cat_reid","epochs":500,"learning_rate":0.00008,"batch_size":24,"freeze_epochs":10,"prepare_data":false,"auto_activate":true,"training_location":"local","household_id":1}'
```

### Training Job Summary

| Batch | LR | Epochs | Freeze | Rationale |
|-------|------|--------|--------|-----------|
| 1a | 0.0001 | 300 | 5 | Re-run best (Rank-1=0.9599) |
| 1b | 0.0001 | 300 | 5 | Re-run #2 (Rank-1=0.9531) |
| 1c | 0.0001 | 250 | 5 | Re-run #3 (Rank-1=0.9270) |
| 2a | 0.0001 | 500 | 10 | Best config scaled up |
| 2b | 0.0001 | 500 | 15 | #2 scaled with longer warmup |
| 2c | 0.0001 | 400 | 10 | #3 scaled up |
| 3a | 0.00015 | 400 | 15 | Higher LR + longer freeze |
| 3b | 0.0001 | 600 | 20 | Sweet-spot LR, max epochs |
| 3c | 0.00008 | 500 | 10 | Slightly lower LR, long training |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guidorugo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/guidorugo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
