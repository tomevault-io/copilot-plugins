## etsy-collages

> **(Inherited from global AGENTS.md)**

# Collage_ETSY - Agent Instructions

# SYSTEM ROLE & BEHAVIORAL PROTOCOLS

**(Inherited from global AGENTS.md)**
- **Role:** Senior Frontend Architect & Avant-Garde UI Designer.
- **Philosophy:** Intentional Minimalism.

---

## Project Overview

**Collage_ETSY** is a specialized tool for generating collages of yarn images. It processes images from an input folder, groups them by product/color, and generates a combined image suitable for Etsy listings.

### Key Features
- **Input:** Images in `input/` directory.
- **Processing:** Groups images by filename analysis (e.g., product name and color code).
- **Output:** Saves collages to `output/` and logs processed file stats.
- **Deduplication:** Automatically handles and removes duplicate "b-side" images if configured.

---

## 🚀 Quick Start

### 1. Environment Setup

```bash
cd "/Users/vostos/Dev/Phtos from websites/Collage_ETSY"
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Running the Script

Place your source images into the `input/` folder, then run:

```bash
python3 main.py
```

The script will:
1. Scan `input/`.
2. Generate collages.
3. Save results to `output/`.

---

## 🧪 TDD Policy

> [!IMPORTANT]
> **This project follows the workspace-wide TDD policy.**

1.  **Test First:** Before adding new logic (e.g., new grouping regex), write a test case.
2.  **Run Tests:** `pytest` (if tests are present).
3.  **Refactor:** Keep `main.py` clean and readable.

---

## 📂 File Structure

```
Collage_ETSY/
├── input/           # PLACE RAW IMAGES HERE
├── output/          # GENERATED COLLAGES APPEAR HERE
├── main.py          # Core logic script
├── requirements.txt # Python dependencies (Pillow)
└── .venv/           # Virtual environment
```

## Critical Rules

1. **Input Hygiene:** Always clear `input/` before starting a new batch to avoid mixing products.
2. **Output Handling:** The script may overwrite files in `output/` if filenames match.
3. **Image naming:** The script relies on filenames ending in digits for color codes (e.g., `Product 101.jpg`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vostos007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Vostos007)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
