## tajweeddsp

> TajweedPro Tutor is a hybrid web application designed to teach and correct Quran recitation (Tajweed) using Digital Signal Processing (DSP) and AI. It replaces generic STT (Speech-to-Text) with physics-based audio analysis to verify specific Tajweed rules like Madd (elongation) and Qalqalah (echoing).

# TajweedPro Tutor - Project Context

## Project Overview

TajweedPro Tutor is a hybrid web application designed to teach and correct Quran recitation (Tajweed) using Digital Signal Processing (DSP) and AI. It replaces generic STT (Speech-to-Text) with physics-based audio analysis to verify specific Tajweed rules like Madd (elongation) and Qalqalah (echoing).

**Key Technologies:**
*   **Frontend:** React 19, Vite, TypeScript, Lucide React.
*   **Backend:** Python 3.12, Flask, Flask-CORS.
*   **DSP Engine:** `librosa`, `numpy`, `scipy`.
*   **Optimization:** `optuna` for hyperparameter tuning.

## Architecture

1.  **Frontend (`App.tsx`, `services/tajwidService.ts`):**
    *   Captures user audio.
    *   Sends audio blob + metadata (target Ayah) to the backend API.
    *   Displays real-time feedback (score, errors, coaching tips).

2.  **Backend (`app.py`):**
    *   Flask server running on port 5000.
    *   Endpoint: `POST /analyze`.
    *   Parses metadata to determine which Tajweed rules apply to the specific text.
    *   Delegates audio analysis to `TajweedDSP`.

3.  **DSP Engine (`dsp_engine.py`):**
    *   **Stateless logic:** Analyzes audio files based on provided configuration.
    *   **Verifiers:**
        *   `check_madd`: Ratio of vowel duration to speech rate.
        *   `check_qalqalah`: Energy envelope burst detection.
        *   `check_ghunnah`: Spectral centroid/rolloff for nasal sounds.
        *   `check_tafkhim`: Formant analysis (F2 frequency) for heavy letters.
        *   `check_hamzah`: Onset strength and silence ratio for glottal stops.

## Development Workflow

### Prerequisites
*   Node.js (v18+)
*   Python 3.12+
*   Python packages: `flask`, `flask-cors`, `librosa`, `numpy`, `scipy`, `optuna`, `pandas`, `openpyxl`, `huggingface_hub`.

### Setup

1.  **Frontend:**
    ```bash
    npm install
    ```

2.  **Backend & Data:**
    ```bash
    # Install dependencies (ensure --user if needed)
    pip install flask flask-cors librosa numpy scipy optuna pandas openpyxl huggingface_hub

    # Initialize data (Downloads datasets and sorts them)
    python setup_data.py
    ```

### Tuning (Optional)
To optimize the DSP thresholds based on the downloaded dataset:
```bash
python tune_dsp.py
```
This generates `final_tajweed_config.json`.

### Running the Application

1.  **Start Backend:**
    ```bash
    python app.py
    ```
    (Runs on `http://127.0.0.1:5000`)

2.  **Start Frontend:**
    ```bash
    npm run dev
    ```
    (Runs on `http://localhost:5173`)

## Key Files Dictionary

*   **`app.py`**: Main entry point for the Flask backend. Maps routes to DSP logic.
*   **`dsp_engine.py`**: Core signal processing logic. Contains the `TajweedDSP` class.
*   **`tune_dsp.py`**: ML-Ops script using Optuna to find the best thresholds for DSP rules.
*   **`setup_data.py`**: Data engineering script. Downloads reference audio, cleans it, and sorts it into `data/processed`.
*   **`services/tajwidService.ts`**: Frontend service layer that bridges React and the Flask API.
*   **`final_tajweed_config.json`**: JSON file storing the optimized threshold values.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MohammadHoukan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MohammadHoukan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
