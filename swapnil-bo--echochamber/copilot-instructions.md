## echochamber

> Handles: PDF file path, Wikipedia URL, any HTTP URL, raw pasted text.

# CLAUDE.md — EchoChamber: Auto-Podcast Generator
## Master Build Prompt for Claude Code

---

## 🎯 Project Overview

**EchoChamber** is a full-stack AI pipeline that transforms any URL, PDF, or Wikipedia page into a 5-minute, two-host audio podcast. Two AI personas (a skeptical host and an enthusiastic expert) debate the content, producing a structured script that is synthesized into audio using two distinct TTS voices, stitched together with intro/outro music, and served via a sleek React frontend with a real-time progress tracker.

This is a **portfolio project** targeting AI/ML Product Management roles. Code must be clean, well-commented, and structured so every architectural decision is explainable in an interview.

---

## 📁 Project Structure

```
echochamber/
├── backend/
│   ├── main.py                    # FastAPI app, CORS, startup hooks, health endpoint
│   ├── routers/
│   │   └── podcast.py             # /generate and /status/{job_id} endpoints
│   ├── pipeline/
│   │   ├── extractor.py           # Text extraction (URL, PDF, Wikipedia)
│   │   ├── chunker.py             # LangChain text chunking + summarization
│   │   ├── script_generator.py    # Gemini Flash — generates JSON script
│   │   ├── tts_engine.py          # edge-tts async TTS per line with retry
│   │   └── audio_mixer.py         # pydub stitching, normalization, fade
│   ├── utils/
│   │   ├── cache.py               # Script JSON cache + per-line TTS cache
│   │   └── job_store.py           # In-memory job status store
│   ├── static/
│   │   ├── intro.mp3              # Royalty-free intro music (pixabay.com)
│   │   └── outro.mp3              # Royalty-free outro music (pixabay.com)
│   ├── outputs/                   # Generated final podcast .mp3 files (gitignored)
│   ├── cache/
│   │   ├── scripts/               # Cached JSON scripts keyed by content hash (gitignored)
│   │   └── tts_lines/             # Cached per-line .mp3 files keyed by hash (gitignored)
│   ├── requirements.txt
│   ├── build.sh                   # Render build script — installs ffmpeg
│   └── .env
├── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── components/
│   │   │   ├── InputForm.jsx          # URL/PDF/Wiki input with auto-detection
│   │   │   ├── ProgressTracker.jsx    # Glass-box live pipeline status
│   │   │   ├── AudioPlayer.jsx        # HTML5 audio player + onError handler
│   │   │   └── Transcript.jsx         # Script preview below audio player
│   │   └── index.css
│   ├── package.json
│   └── vite.config.js
├── .gitignore
└── CLAUDE.md
```

---

## 🛠️ Final Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| Web Scraping | `requests` + `BeautifulSoup4` | |
| PDF Parsing | `PyMuPDF` (fitz) | |
| Wikipedia | `wikipedia-api` | |
| Text Chunking | `LangChain` text splitters + map_reduce | Hard cap at 1500 words |
| LLM | `Gemini 2.0 Flash` via `google-generativeai` | Free tier, 1500 req/day |
| TTS Voice A — Skeptic Host | `edge-tts` → `en-US-GuyNeural` | Async |
| TTS Voice B — Expert | `edge-tts` → `en-US-JennyNeural` | Async |
| TTS Retry | `tenacity` | 3 attempts, exponential backoff |
| Audio Stitching | `pydub` + `ffmpeg` | |
| Async CPU Offload | `asyncio.to_thread` | Prevents event loop blocking |
| Backend | `FastAPI` + `asyncio` | Async job pipeline |
| Job State | In-memory dict (`job_store.py`) | No Redis needed |
| Frontend | `React` + `Vite` + `TailwindCSS` | |
| Frontend Deploy | `Vercel` | |
| Backend Deploy | `Render` free tier (512MB RAM) | |

---

## ⚙️ Backend Implementation

---

### `main.py` — App Entry Point

This file handles four critical responsibilities:

**1. CORS Middleware (required for Vercel ↔ Render communication)**

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        os.getenv("ALLOWED_ORIGINS", "http://localhost:5173"),  # Vercel prod URL
        "http://localhost:5173",                                 # Local dev
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Without this, the browser will silently block all requests from the Vercel frontend to the Render backend due to CORS policy. This is non-negotiable.

**2. Startup Directory Initialization**

```python
@app.on_event("startup")
async def startup_event():
    # Create required directories if they don't exist.
    # These are gitignored and won't exist on a fresh clone or Render deploy.
    os.makedirs("outputs", exist_ok=True)
    os.makedirs("cache/scripts", exist_ok=True)
    os.makedirs("cache/tts_lines", exist_ok=True)
```

Without this, the first generation attempt will crash with `FileNotFoundError` on a fresh Git clone or Render deploy.

**3. Health Check Endpoint (Render cold start UX fix)**

```python
@app.get("/health")
async def health():
    return {"status": "ok"}
```

The React frontend pings this before starting a generation. If Render has been idle for 15+ minutes, the instance needs 30–50 seconds to boot. The frontend detects a slow or failed health check and shows a user-friendly "Backend waking up..." message instead of silently hanging.

**4. Static file serving**

```python
from fastapi.staticfiles import StaticFiles
app.mount("/audio", StaticFiles(directory="outputs"), name="audio")
```

---

### `routers/podcast.py` — API Endpoints

**POST `/generate`**
- Accepts `{ "input": "https://..." }`
- Creates a UUID job ID
- Fires async background task via `asyncio` — never blocks
- Returns `{ "job_id": "uuid" }` immediately to the frontend

**GET `/status/{job_id}`**
- Returns current job state from `job_store`
- Frontend polls this every 2 seconds
- Returns one of: `queued` / `processing` / `complete` / `failed`
- On `complete`: also returns `{ "audio_url": "...", "script": [...] }`

**Pipeline status messages emitted in order:**
1. `"Detecting input type..."`
2. `"Extracting text..."`
3. `"Chunking and summarizing content..."`
4. `"Checking script cache..."`
5. `"Agents debating (Gemini Flash)..."`
6. `"Synthesizing audio (Line X of Y)..."` — updated per line
7. `"Mixing and mastering audio..."`
8. `"Podcast ready!"`

---

### `extractor.py` — Auto-Detecting Input Type

```python
def extract_text(user_input: str) -> str:
    """
    Auto-detects input type so the user never has to configure anything.
    Handles: PDF file path, Wikipedia URL, any HTTP URL, raw pasted text.
    """
    if user_input.lower().endswith(".pdf"):
        return extract_pdf(user_input)
    elif "wikipedia.org" in user_input:
        return extract_wikipedia(user_input)
    elif user_input.startswith("http://") or user_input.startswith("https://"):
        return extract_url(user_input)
    else:
        return user_input  # treat as raw pasted text
```

- `extract_url`: Use `requests` + `BeautifulSoup`. Extract `<p>` tags only. Strip `<script>`, `<nav>`, `<footer>`.
- `extract_pdf`: Use `PyMuPDF` (`fitz`). Loop all pages, extract text blocks, join with newlines.
- `extract_wikipedia`: Use `wikipedia-api` lib. Return `.summary + "\n\n" + .content`.
  **CRITICAL: Must initialize with a proper User-Agent or Wikipedia returns 403 Forbidden:**
  ```python
  import wikipediaapi
  wiki = wikipediaapi.Wikipedia(
      user_agent='EchoChamber/1.0 (your-email@gmail.com)',
      language='en'
  )
  ```
  Wikipedia's Wikimedia API strictly enforces User-Agent policy. Default initialization with no user_agent will be blocked on the first request.

---

### `chunker.py` — Text Normalization

- Use `LangChain RecursiveCharacterTextSplitter` with `chunk_size=2000`, `chunk_overlap=200`
- Count words after splitting
- If word count exceeds 1500: use LangChain `load_summarize_chain` with `chain_type="map_reduce"` and Gemini Flash as the LLM to compress to ~1500 words
- This handles the 50-page PDF edge case cleanly without crashing the pipeline

---

### `script_generator.py` — Single Gemini Call

**Critical design decision: one structured Gemini call, not two separate agent calls.**

Reasons: 50% less API quota used, more coherent dialogue (model sees both sides simultaneously), simpler error handling.

```python
import google.generativeai as genai
from google.generativeai.types import HarmCategory, HarmBlockThreshold

# BLOCK_ONLY_HIGH — not BLOCK_NONE
# Handles controversial/political articles while blocking genuinely harmful content.
# BLOCK_NONE is a security smell in public-facing portfolio apps.
SAFETY_SETTINGS = {
    HarmCategory.HARM_CATEGORY_HARASSMENT: HarmBlockThreshold.BLOCK_ONLY_HIGH,
    HarmCategory.HARM_CATEGORY_HATE_SPEECH: HarmBlockThreshold.BLOCK_ONLY_HIGH,
    HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT: HarmBlockThreshold.BLOCK_ONLY_HIGH,
    HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: HarmBlockThreshold.BLOCK_ONLY_HIGH,
}
```

**System Prompt — inject into every Gemini call:**

```
You are a podcast script writer. Given the following article text, generate a
lively, argumentative 5-minute podcast dialogue between two hosts:

- HOST (Alex): Skeptical, analytical, devil's advocate. Challenges every claim.
- EXPERT (Maya): Enthusiastic, optimistic, deeply knowledgeable about the subject.

Rules:
1. Generate exactly 16–20 dialogue turns total, strictly alternating HOST then EXPERT.
2. Each turn must be 2–4 sentences. Use natural spoken language only.
3. Start with a 2-sentence intro from HOST setting up the topic for the listener.
4. End with HOST and EXPERT reaching a nuanced, surprising agreement.
5. Optimize the dialogue for Text-to-Speech engines:
   - Write all acronyms with dashes: L-L-M, R-A-G, A-P-I, G-P-T, F-A-S-T-A-P-I
   - Write numbers as words: "forty-two percent" not "42%"
   - No markdown, bullet points, asterisks, or formatting characters anywhere.

Return ONLY a valid JSON array. No preamble, no explanation, no markdown code fences.
Format:
[
  {"speaker": "HOST", "line": "..."},
  {"speaker": "EXPERT", "line": "..."}
]
```

**Error handling:**

```python
try:
    response = model.generate_content(prompt, safety_settings=SAFETY_SETTINGS)
    script = json.loads(response.text)
    return script
except google.generativeai.types.StopCandidateException:
    raise ValueError("Content blocked by AI safety filters. Try a different article.")
except json.JSONDecodeError:
    raise ValueError("Script generation returned invalid format. Please retry.")
```

---

### `tts_engine.py` — Async TTS with Retry + Per-Line Cache

```python
import edge_tts
import hashlib
import os
from tenacity import retry, stop_after_attempt, wait_exponential

VOICE_MAP = {
    "HOST": "en-US-GuyNeural",
    "EXPERT": "en-US-JennyNeural"
}

TTS_CACHE_DIR = "cache/tts_lines"

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=8))
async def generate_line_audio(speaker: str, line: str) -> str:
    """
    Generates TTS for a single script line.
    Checks per-line cache first — only calls edge-tts on a cache miss.
    Returns the path to the generated .mp3 file.
    tenacity handles edge-tts connection drops (3 attempts, exponential backoff).
    """
    cache_key = hashlib.md5(f"{speaker}:{line}".encode()).hexdigest()
    output_path = f"{TTS_CACHE_DIR}/{cache_key}.mp3"

    if os.path.exists(output_path):
        return output_path  # Cache hit — skip edge-tts entirely

    voice = VOICE_MAP[speaker]
    communicate = edge_tts.Communicate(line, voice)
    await communicate.save(output_path)
    return output_path
```

**Per-line cache rationale:** During Days 6-10 (audio tuning), you'll re-run the stitching pipeline constantly. Without this, all 18 TTS lines regenerate every run. With it, only changed lines regenerate. Iteration becomes near-instant.

---

### `audio_mixer.py` — Stitching Pipeline

```python
from pydub import AudioSegment
from pydub import effects
import os

SILENCE_BETWEEN_LINES = 500   # ms — prevents robotic, rushed feel
INTRO_FADE_DURATION   = 3000  # ms — music fades under first spoken line
OUTRO_FADE_DURATION   = 3000  # ms

def mix_podcast(script: list, line_audio_paths: list, output_path: str):
    """
    CRITICAL: This function is CPU-bound and synchronous.
    Always call via: await asyncio.to_thread(mix_podcast, script, paths, output_path)
    Never call directly inside a FastAPI async context — it will block the event
    loop for 10-20 seconds and cause the /status polling to hang and timeout.
    """
    intro = AudioSegment.from_mp3("static/intro.mp3")
    outro = AudioSegment.from_mp3("static/outro.mp3")
    silence = AudioSegment.silent(duration=SILENCE_BETWEEN_LINES)

    # Fade intro music out over 3 seconds for a professional feel
    podcast_audio = intro[:6000].fade_out(INTRO_FADE_DURATION)

    for audio_path in line_audio_paths:
        line_audio = AudioSegment.from_mp3(audio_path)

        # CRITICAL: Normalize sample rate on every line before appending.
        # edge-tts voices output different sample rates (e.g. 16kHz vs 24kHz).
        # Without this, pydub stitches silently but certain lines play warped or off-speed.
        line_audio = (
            line_audio
            .set_frame_rate(24000)
            .set_channels(1)
            .set_sample_width(2)
        )

        podcast_audio += line_audio + silence

        # Free RAM by explicitly deleting the in-memory pydub object after appending.
        # Python's GC handles this at end of loop scope, but del is explicit and clear.
        # CRITICAL: Do NOT use os.remove(audio_path) here — that destroys the per-line
        # TTS cache in cache/tts_lines/, causing every re-run to be a cache miss.
        # The file on disk IS the cache. Only the in-memory object needs to be released.
        del line_audio

    podcast_audio += outro.fade_out(OUTRO_FADE_DURATION)

    # Normalize final track — balances volume between HOST and EXPERT voices.
    # edge-tts voice models have different natural loudness levels.
    podcast_audio = effects.normalize(podcast_audio)

    podcast_audio.export(output_path, format="mp3")
```

**Calling this correctly from the background task:**

```python
# In routers/podcast.py — CORRECT way to call mix_podcast from async context
import asyncio

await asyncio.to_thread(mix_podcast, script, line_audio_paths, output_path)
```

---

### `cache.py` — Script-Level Caching

```python
import hashlib, json, os

SCRIPT_CACHE_DIR = "cache/scripts"

def get_cache_key(text: str) -> str:
    return hashlib.md5(text.encode()).hexdigest()

def get_cached_script(text: str):
    """Returns cached script JSON if it exists, else None."""
    path = f"{SCRIPT_CACHE_DIR}/{get_cache_key(text)}.json"
    if os.path.exists(path):
        with open(path) as f:
            return json.load(f)
    return None

def save_script_cache(text: str, script: list):
    """Saves generated script JSON keyed by content hash."""
    os.makedirs(SCRIPT_CACHE_DIR, exist_ok=True)
    with open(f"{SCRIPT_CACHE_DIR}/{get_cache_key(text)}.json", "w") as f:
        json.dump(script, f)
```

**Script cache rationale:** Saves Gemini API quota during development. Re-running the same article skips the LLM call entirely. Essential for conserving the 1500 req/day free tier during active testing.

---

### `job_store.py` — In-Memory Job State

```python
# Simple in-memory store — no Redis or database needed for this scale
jobs = {}

def create_job(job_id: str):
    jobs[job_id] = {
        "status": "queued",
        "message": "Job queued...",
        "result": None,
        "error": None
    }

def update_job(job_id: str, message: str, status: str = "processing"):
    jobs[job_id]["status"] = status
    jobs[job_id]["message"] = message

def complete_job(job_id: str, audio_url: str, script: list):
    jobs[job_id]["status"] = "complete"
    jobs[job_id]["message"] = "Podcast ready!"
    jobs[job_id]["result"] = {"audio_url": audio_url, "script": script}

def fail_job(job_id: str, error: str):
    jobs[job_id]["status"] = "failed"
    jobs[job_id]["error"] = error
```

---

## 🎨 Frontend Implementation

---

### `App.jsx` — Application State

Manages three views: `input` → `loading` → `result`

On mount, ping `GET /health`. If response is slow (>3s) or fails, show banner:
> *"Backend is waking up from sleep. This may take 30 seconds on first load..."*

This handles Render's free tier cold start (spins down after 15 minutes idle). Without it, the frontend silently hangs and looks broken to any observer during a live demo.

---

### `InputForm.jsx`

- Single text input accepting URL, Wikipedia link, or PDF file path
- "Generate Podcast" button
- On submit: POST to `/generate`, store `job_id`, switch to progress view
- Auto-detection logic lives entirely in the backend — frontend just passes the raw input string

---

### `ProgressTracker.jsx` — Glass Box UX

**This component is the biggest UX differentiator in the entire project.**

- Polls `GET /status/{job_id}` every 2 seconds using `setInterval`
- Renders each pipeline stage as a checklist with ✅ on completion
- Shows the current active stage as animated pulsing text
- "Synthesizing audio (Line 4 of 18)..." updates live — user sees real progress
- On `status === "complete"`: clear interval, render `AudioPlayer` + `Transcript`
- On `status === "failed"`: clear interval, show error message with a "Try Again" button

This turns a 45-second wait into a transparent, engaging experience. It signals production-minded thinking — not just a loading spinner.

---

### `AudioPlayer.jsx`

```jsx
// Native HTML5 audio — zero config, zero bundle cost, clean with Tailwind.
// wavesurfer.js was explicitly rejected: too heavy, requires backend waveform
// preprocessing, and adds complexity with no meaningful UX gain for this project.

<audio
  controls
  src={audioUrl}
  className="w-full rounded-lg"
  onError={() => {
    // Handles Render ephemeral disk wipe gracefully.
    // Render free tier deletes all generated files when the instance spins down.
    // Without this, the <audio> element throws a raw browser error
    // and the UI silently breaks with no feedback to the user.
    setAudioError("Audio session expired. Please regenerate the podcast.");
  }}
/>
{audioError && (
  <p className="text-red-500 text-sm mt-2">{audioError}</p>
)}
```

---

### `Transcript.jsx`

- Renders the `script` array returned in `/status/{job_id}` result
- HOST lines: left-aligned, gray text, bold "Alex:" label
- EXPERT lines: left-aligned, blue text, bold "Maya:" label
- Displayed below the audio player
- Zero extra backend work — the JSON script is already in the job result payload
- In a portfolio demo or interview walkthrough, this makes the app dramatically more impressive

---

## 🌱 Environment Variables

**Backend `.env`:**
```
GEMINI_API_KEY=your_key_here
OUTPUT_DIR=outputs
CACHE_DIR=cache
ALLOWED_ORIGINS=https://your-app.vercel.app
```

**Frontend (Vercel dashboard env vars):**
```
VITE_API_BASE_URL=https://your-api.onrender.com
```

---

## 📦 `requirements.txt`

```
fastapi
uvicorn
python-dotenv
requests
beautifulsoup4
PyMuPDF
wikipedia-api
langchain
langchain-google-genai
google-generativeai
edge-tts
pydub
tenacity
```

---

## 📄 `.gitignore`

```
# Environment
.env

# Generated outputs — never commit audio files or cached scripts to Git
outputs/
cache/
*.mp3

# Python
__pycache__/
*.pyc
*.pyo
.venv/
venv/

# Node
node_modules/
dist/

# OS
.DS_Store
Thumbs.db
```

---

## 🚀 Deployment

### Render (Backend)

**`build.sh`** — ffmpeg is pre-installed on Render's native Python environments. Do NOT add apt-get installs — Render does not grant root access in build scripts and it will fail with Permission Denied:
```bash
#!/bin/bash
pip install -r requirements.txt
```

- Set all `.env` variables in the Render dashboard environment section
- The `@app.on_event("startup")` block handles directory creation automatically on every deploy
- `outputs/` and `cache/` are ephemeral — wiped when Render spins down. This is expected behavior, handled gracefully by the frontend `onError` handler
- Explicitly releasing the pydub object (`del line_audio`) after each line append in `audio_mixer.py` is non-negotiable for staying within 512MB RAM

### Vercel (Frontend)

- Set `VITE_API_BASE_URL=https://your-render-url.onrender.com` in Vercel environment variables
- Standard Vite + React deploy, no special config needed
- The `/health` pre-flight check in `App.jsx` handles Render cold starts gracefully

---

## 🧪 Testing Checklist (Days 14–15)

**Pipeline:**
- [ ] Short URL (< 500 words) — happy path baseline
- [ ] Long URL (> 3000 words) — tests chunker + map_reduce summarization
- [ ] Wikipedia URL — tests `wikipedia-api` extractor
- [ ] PDF file path — tests `PyMuPDF` extractor
- [ ] Controversial/political article — tests `BLOCK_ONLY_HIGH` + `StopCandidateException` handler
- [ ] AI/ML article with acronyms (LLM, RAG, API, FastAPI) — tests TTS acronym prompt fix
- [ ] Same URL submitted twice — confirms script-level cache hit (no Gemini call fired)
- [ ] Same script re-run for audio — confirms per-line TTS cache hit (no edge-tts calls fired)

**Audio:**
- [ ] No warping between HOST and EXPERT lines — confirms sample rate normalization
- [ ] Balanced volume between voices — confirms `pydub.effects.normalize()`
- [ ] Intro music fades smoothly under first spoken line
- [ ] 500ms silence between speaker turns (not robotic)
- [ ] Outro fades out cleanly

**Frontend:**
- [ ] Progress tracker updates for every pipeline stage
- [ ] "Synthesizing audio (Line X of Y)" counter updates per line
- [ ] Transcript renders with correct HOST/EXPERT color styling
- [ ] `onError` on `<audio>` shows graceful expired message (test by deleting output file manually)
- [ ] Cold start banner appears when Render instance is sleeping (test after 15min idle)

**Deployment:**
- [ ] Fresh Git clone → startup hook creates all required directories automatically
- [ ] Vercel frontend successfully polls Render backend (CORS not blocking)
- [ ] Render deploy: no OOM kill during audio mixing step (RAM stays under 512MB)
- [ ] `ffmpeg` available on Render after `build.sh` runs

---

## ⚠️ Complete Edge Case Registry

| Edge Case | Root Cause | Fix Location | Fix |
|---|---|---|---|
| Article too long (50-page PDF) | Exceeds Gemini context window | `chunker.py` | LangChain map_reduce → hard cap 1500 words |
| Controversial article safety block | Gemini default safety thresholds | `script_generator.py` | `BLOCK_ONLY_HIGH` + `StopCandidateException` catch |
| edge-tts random connection drop | Reverse-engineered API instability | `tts_engine.py` | `tenacity` — 3 retries, exponential backoff |
| One voice louder than the other | Different natural loudness per voice model | `audio_mixer.py` | `pydub.effects.normalize()` on final track |
| Audio warping on certain lines | edge-tts voices output different sample rates | `audio_mixer.py` | `.set_frame_rate(24000).set_channels(1).set_sample_width(2)` |
| Render OOM during audio mixing | pydub loads all line files into memory simultaneously | `audio_mixer.py` | `del line_audio` after each append — releases in-memory object, preserves cache file on disk |
| Acronyms mispronounced (RAG, LLM) | TTS reads acronyms as words | `script_generator.py` | System prompt: write `R-A-G`, `L-L-M`, `A-P-I` |
| Gemini quota exhausted during dev | Repeated identical API calls during iteration | `cache.py` | Script-level JSON cache keyed by content hash |
| Per-line TTS being regenerated | No caching of individual audio lines | `tts_engine.py` | Per-line `.mp3` cache keyed by `hash(speaker + line)` |
| 45-second blank loading screen | No pipeline visibility | `ProgressTracker.jsx` | Granular status messages emitted per stage via polling |
| pydub blocks FastAPI event loop | CPU-bound sync function in async context | `routers/podcast.py` | `asyncio.to_thread(mix_podcast, ...)` |
| Vercel requests blocked by Render | Cross-origin requests blocked by default | `main.py` | `CORSMiddleware` with Vercel + localhost origins |
| `FileNotFoundError` on fresh deploy | Gitignored dirs don't exist on clone or Render | `main.py` | `@app.on_event("startup")` runs `os.makedirs()` |
| Ephemeral 404 after Render sleep | Render wipes disk on instance spin-down | `AudioPlayer.jsx` | `onError` → "Audio session expired. Please regenerate." |
| Accidentally committing audio to Git | No `.gitignore` for outputs and cache | `.gitignore` | `outputs/`, `cache/`, `*.mp3` all gitignored |
| Silent frontend hang on cold start | Render takes 30-50s to wake from 15min idle | `App.jsx` | `/health` pre-flight + "Backend waking up..." banner |
| Wikipedia API 403 Forbidden | Wikimedia enforces User-Agent policy, default client is blocked | `extractor.py` | Initialize with `wikipediaapi.Wikipedia(user_agent='EchoChamber/1.0 (email)', language='en')` |

---

## 💡 Interview Talking Points (Know These Cold)

**1. Why one Gemini call instead of two separate agents?**
Reduces API quota usage by 50%, produces more coherent dialogue since the model sees both sides of the conversation simultaneously, and simplifies error handling to a single try/except block.

**2. Why `BLOCK_ONLY_HIGH` instead of `BLOCK_NONE`?**
Production-safety mindset. `BLOCK_NONE` is a security smell in public-facing apps — it signals carelessness to anyone reviewing the code. `BLOCK_ONLY_HIGH` handles real-world controversial content while still blocking genuinely harmful outputs. More defensible in any code review or interview.

**3. Why cache at both script level AND per-line TTS level?**
Two different development phases need two different caches. Script cache saves Gemini quota during audio tuning (Days 6-10). Line cache saves edge-tts calls when iterating on stitching and normalization logic. Together they make re-runs near-instant without touching any external API.

**4. Why `asyncio.to_thread` for pydub?**
FastAPI runs on an async event loop. pydub is synchronous and CPU-bound — running it directly would freeze the event loop for 10-20 seconds and cause every frontend polling request to hang and timeout. `asyncio.to_thread` offloads it to a thread pool, keeping the event loop free and responsive.

**5. Why explicitly release the in-memory audio object (`del line_audio`)?**
Render free tier is 512MB RAM. pydub processes audio in memory. Loading 18+ TTS lines plus background music simultaneously would exceed the limit and trigger a silent OOM container restart. Explicitly deleting the object from memory keeps the RAM footprint O(1) instead of O(n), while preserving the actual `.mp3` cache file on disk for rapid re-runs.

**6. Why not wavesurfer.js?**
Over-engineered for this use case. Requires backend waveform preprocessing to look good. Native HTML5 `<audio>` with Tailwind achieves the same UX goal with zero config, zero bundle overhead, and a fraction of the implementation time.

**7. Why a `/health` endpoint and pre-flight check?**
Render free tier spins down after 15 minutes of inactivity. Cold start takes 30-50 seconds. Without the health check, the user clicks Generate and the frontend silently hangs — to any observer it looks like a broken app. The pre-flight turns a confusing failure into a transparent, expected wait state.

**8. Why `BLOCK_ONLY_HIGH` over `BLOCK_MEDIUM_AND_ABOVE`?**
This is a content-first application — users will deliberately feed it controversial articles. `BLOCK_MEDIUM_AND_ABOVE` would reject too many valid inputs and frustrate users. `BLOCK_ONLY_HIGH` strikes the right balance between utility and responsibility for this specific use case.

---
> Source: [Swapnil-bo/EchoChamber](https://github.com/Swapnil-bo/EchoChamber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
