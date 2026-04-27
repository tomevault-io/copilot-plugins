## nutriscan

> NutriScan is an AI-powered nutrition assistant that combats food insecurity through personalized dietary insights. It reads nutrition labels (vision AI, with OCR as offline fallback), identifies food from photos (vision AI), generates nutritious recipes from available ingredients, and connects users to free/low-income food resources nearby. Built by 3 Purdue freshmen (Aarav, Neil, Nuv) through Dataception for the undergraduate research symposium on **April 16, 2026**. All-free tech stack.

# NutriScan Build Plan

## Context
NutriScan is an AI-powered nutrition assistant that combats food insecurity through personalized dietary insights. It reads nutrition labels (vision AI, with OCR as offline fallback), identifies food from photos (vision AI), generates nutritious recipes from available ingredients, and connects users to free/low-income food resources nearby. Built by 3 Purdue freshmen (Aarav, Neil, Nuv) through Dataception for the undergraduate research symposium on **April 16, 2026**. All-free tech stack.

## Tech Stack
- **Python** + **Streamlit** (pure Python frontend)
- **Groq Vision API** with Llama 4 Scout `meta-llama/llama-4-scout-17b-16e-instruct` — **primary** label reader and food photo identifier. Takes an image, returns structured JSON (nutrition fields / food items). Replaced the original Tesseract OCR path for Upload Label and Recipe "Add from Label" flows after Phase 4.2 testing showed OCR was unreliable on real iPhone photos.
- **Tesseract** via `pytesseract` + **OpenCV** — offline **fallback** for the label reader when `GROQ_API_KEY` is missing or the vision API errors out. Kept intact so the app degrades gracefully, not used on the golden path.
- **Groq Text API** with Llama 3.3-70b-versatile — LLM analysis of nutrition data + health profile → allergen/preservative/nutrient flags, goal alignment, recommendations (free tier)
- **USDA FoodData Central API** (free key from api.data.gov) — primary source for raw/whole foods
- **Open Food Facts API** (no key, no rate limits) — fallback for branded packaged products that USDA misses
- **pillow-heif** — HEIC/HEIF support so iPhone photos open natively in the file uploader without client-side conversion
- **Food resource locator API** (TBD — USDA Food Desert Atlas, FoodFinder, Feeding America, or 211.org; fallback: curated local list)
- API keys in `.env` (gitignored), `.env.example` committed for teammates

## Project Structure
```
NutriScan/
├── app.py                      # Streamlit entry point
├── requirements.txt
├── .env.example                # GROQ_API_KEY=  USDA_API_KEY=
├── .env                        # actual keys (gitignored)
├── .gitignore
├── src/
│   ├── ocr/                    # Fallback label reader (Phase 3.1)
│   │   ├── preprocessor.py     # OpenCV image preprocessing (EXIF, normalize width, adaptive threshold)
│   │   └── extractor.py        # Tesseract OCR + regex parsing
│   ├── llm/
│   │   ├── groq_client.py      # Groq API wrapper
│   │   └── prompts.py          # Prompt templates (analysis, recipe, food vision, label vision)
│   ├── nutrition/
│   │   ├── models.py           # Dataclasses
│   │   ├── fda_guidelines.py   # DV% computation
│   │   ├── usda_client.py      # USDA API client + lookup_food fallback wrapper
│   │   └── openfoodfacts_client.py  # Open Food Facts API client (fallback)
│   ├── vision/                 # Primary label + food readers (Phase 3.4, Phase 4.2)
│   │   ├── food_identifier.py  # Groq vision → food items + USDA/OFF bridge
│   │   └── label_reader.py     # Groq vision → NutritionData (replaces OCR on golden path)
│   ├── resources/
│   │   └── locator.py          # Free food resource finder (food banks, pantries, etc.)
│   └── ui/
│       ├── components.py       # Reusable Streamlit widgets
│       ├── pages_upload.py     # Image upload tab
│       ├── pages_snap.py       # Snap Food photo tab
│       ├── pages_manual.py     # Manual entry tab
│       ├── pages_recipe.py     # Recipe Generator tab
│       ├── pages_find.py       # Find Free Food Near You tab
│       └── pages_results.py    # Results display
├── data/
│   └── fda_daily_values.json   # FDA daily reference values
├── tests/
│   ├── test_ocr.py
│   ├── test_llm.py
│   ├── test_nutrition.py
│   └── sample_labels/
└── eval/
    ├── ocr_accuracy.py
    └── ground_truth.json
```

## Key Packages
```
streamlit>=1.30.0
opencv-python-headless>=4.8.0
pytesseract>=0.3.10
Pillow>=10.0.0
pillow-heif>=0.13.0
groq>=0.4.0
requests>=2.31.0
python-dotenv>=1.0.0
pytest>=7.4.0
```
System dependency: `brew install tesseract` (macOS) / `apt install tesseract-ocr` (Ubuntu) — only needed for the OCR fallback path; the primary vision-based label reader has no system dependencies.

---

### Phase 1 — Project Setup `March 31 – April 1`
> Goal: Get a running Streamlit skeleton with all dependencies installed.

- [x] **1.1 Scaffolding**
  - [x] 1.1.1 Create directory structure (all folders and `__init__.py` files)
  - [x] 1.1.2 Create `requirements.txt` with all packages listed above
  - [x] 1.1.3 Create `.gitignore` (include `.env`, `__pycache__/`, `.venv/`, `*.pyc`)
  - [x] 1.1.4 Create `.env.example` with `GROQ_API_KEY=` and `USDA_API_KEY=`
  - [x] 1.1.5 Set up virtual environment and install dependencies
  - [x] 1.1.6 Install Tesseract OCR system dependency

- [x] **1.2 Minimal Streamlit App**
  - [x] 1.2.1 Create `app.py` with page config (`st.set_page_config(page_title="NutriScan", layout="wide")`)
  - [x] 1.2.2 Add title, sidebar placeholder, and three tabs ("Upload Label", "Snap Food", "Manual Entry")
  - [x] 1.2.3 Verify `streamlit run app.py` launches in browser

- [x] **1.3 Git Init**
  - [x] 1.3.1 Initialize git repo
  - [x] 1.3.2 Create initial commit with scaffolding
  - [x] 1.3.3 Push to GitHub (confirm `.env` is not included)

---

### Phase 2 — Data Models + FDA Reference `April 1 – April 2`
> Goal: Build the data backbone that every other module depends on.

- [x] **2.1 Dataclasses** (`src/nutrition/models.py`)
  - [x] 2.1.1 Define `NutritionData` — calories, total_fat, saturated_fat, trans_fat, cholesterol, sodium, total_carbs, dietary_fiber, total_sugars, added_sugars, protein, vitamin_d, calcium, iron, potassium, serving_size, servings_per_container, ingredients_list
  - [x] 2.1.2 Define `HealthProfile` — caloric_target, dietary_goals (list[str]), allergens (list[str]), restrictions (list[str])
  - [x] 2.1.3 Define `AnalysisResult` — allergen_flags, preservative_flags, nutrient_flags, goal_alignment, recommendations, overall_risk, summary

- [x] **2.2 FDA Daily Values** (`data/fda_daily_values.json`)
  - [x] 2.2.1 Create JSON file with FDA 2,000-calorie daily reference values for all nutrients
  - [x] 2.2.2 Source values from FDA.gov (public data)

- [x] **2.3 DV% Computation** (`src/nutrition/fda_guidelines.py`)
  - [x] 2.3.1 Write `load_fda_values()` to read the JSON file
  - [x] 2.3.2 Write `compute_dv_percentages(nutrition_data: NutritionData) -> dict`
  - [x] 2.3.3 Test manually: 20g total fat → ~26% DV (based on 78g reference)

---

### Phase 3 — Core Features — Parallel Tracks `April 2 – April 7`
> Goal: Build the three independent subsystems. Aarav, Neil, and Nuv each take one track.

- [x] **3.1 OCR Pipeline (Track A — Neil)** — now the **offline fallback** for the label reader. The primary path is the Groq vision label reader added in Phase 4.2 ([src/vision/label_reader.py](src/vision/label_reader.py)); this OCR pipeline is only hit when `GROQ_API_KEY` is missing or the vision API errors out.

  - [x] **3.1.1 Image Preprocessing** (`src/ocr/preprocessor.py`)
    - [x] 3.1.1.1 Accept PIL Image or file path as input
    - [x] 3.1.1.2 Convert to grayscale
    - [x] 3.1.1.3 Resize if image is too small (< 300 DPI equivalent)
    - [x] 3.1.1.4 Apply adaptive thresholding (`cv2.adaptiveThreshold` with `ADAPTIVE_THRESH_GAUSSIAN_C`)
    - [x] 3.1.1.5 Apply Gaussian blur to reduce noise
    - [x] 3.1.1.6 Return processed image as numpy array

  - [x] **3.1.2 OCR Extraction** (`src/ocr/extractor.py`)
    - [x] 3.1.2.1 Call `preprocessor.preprocess(image)` to get cleaned image
    - [x] 3.1.2.2 Run `pytesseract.image_to_string()` with config `--psm 6`
    - [x] 3.1.2.3 Build regex patterns for each nutrient field (e.g. `r"total\s*fat\s*(\d+\.?\d*)\s*g"`)
    - [x] 3.1.2.4 Parse ingredients list (find "Ingredients:" line, capture until next section)
    - [x] 3.1.2.5 Return parsed `NutritionData` object + raw OCR text (for debugging)
    - [x] 3.1.2.6 Add confidence indicator — count parsed fields out of ~15; warn if < 5

  - [x] **3.1.3 OCR Testing**
    - [x] 3.1.3.1 Collect 3-4 sample nutrition label photos
    - [x] 3.1.3.2 Print raw Tesseract output to understand actual text before writing regex
    - [x] 3.1.3.3 Write unit tests in `tests/test_ocr.py` with hardcoded strings
    - [x] 3.1.3.4 Test edge cases: missing fields, "8 g" vs "8g", decimal values

- [x] **3.2 LLM Integration (Track B — Aarav)**

  - [x] **3.2.1 Prompt Templates** (`src/llm/prompts.py`)
    - [x] 3.2.1.1 Write system prompt with JSON output schema (allergen detection, preservative flagging, sugar/nutrient flags, goal alignment, recommendations)
    - [x] 3.2.1.2 Define JSON response structure: `allergen_flags`, `preservative_flags`, `nutrient_flags`, `goal_alignment`, `recommendations`, `overall_risk`, `summary`
    - [x] 3.2.1.3 Write user prompt template that fills in nutrition data + DV% + ingredients + health profile

  - [x] **3.2.2 Groq API Client** (`src/llm/groq_client.py`)
    - [x] 3.2.2.1 Load API key from environment variable via `python-dotenv`
    - [x] 3.2.2.2 Initialize Groq client
    - [x] 3.2.2.3 Write `analyze(nutrition_data, health_profile, dv_percentages) -> AnalysisResult`
    - [x] 3.2.2.4 Use `response_format={"type": "json_object"}` to force JSON output
    - [x] 3.2.2.5 Set `temperature=0.3` for factual consistency
    - [x] 3.2.2.6 Parse JSON response into `AnalysisResult` dataclass
    - [x] 3.2.2.7 Add try/except with retry on rate limit (30 RPM, 1K req/day)
    - [x] 3.2.2.8 Show user-friendly error via `st.error()` on failure

  - [x] **3.2.3 LLM Testing**
    - [x] 3.2.3.1 Write unit tests in `tests/test_llm.py` for prompt construction
    - [x] 3.2.3.2 Test response parsing with mock JSON → verify `AnalysisResult`
    - [x] 3.2.3.3 Manual test: call Groq API with sample data, inspect output

- [x] **3.3 USDA API + Streamlit UI (Track C — Nuv)**

  - [x] **3.3.1 USDA Client + Open Food Facts Fallback** (`src/nutrition/usda_client.py`, `src/nutrition/openfoodfacts_client.py`)
    - [x] 3.3.1.1 Register for free API key at api.data.gov
    - [x] 3.3.1.2 Write `search_food(query, api_key) -> dict` calling `/fdc/v1/foods/search`
    - [x] 3.3.1.3 Write `check_preservatives(ingredients_list) -> list[str]` with hardcoded preservative list
    - [x] 3.3.1.4 Cache results in `st.session_state`
    - [x] 3.3.1.5 Write Open Food Facts client (`openfoodfacts_client.py`) + `lookup_food(query, api_key)` wrapper that tries USDA first, falls back to Open Food Facts on miss (covers branded/packaged products USDA doesn't index well)

  - [x] **3.3.2 Health Profile Form** (`src/ui/components.py`)
    - [x] 3.3.2.1 Write `health_profile_form()` for the sidebar (caloric target, allergens multiselect, dietary goals, restrictions)
    - [x] 3.3.2.2 Store profile in `st.session_state`, return `HealthProfile` object

  - [x] **3.3.3 Nutrition Editor** (`src/ui/components.py`)
    - [x] 3.3.3.1 Write `nutrition_editor(nutrition_data)` — editable form pre-filled with OCR data
    - [x] 3.3.3.2 Each nutrient field is an `st.number_input`
    - [x] 3.3.3.3 Text area for ingredients list
    - [x] 3.3.3.4 Return corrected `NutritionData` on submit

  - [x] **3.3.4 Results Display** (`src/ui/pages_results.py`)
    - [x] 3.3.4.1 Write `results_display(result, dv_percentages)`
    - [x] 3.3.4.2 Colored flags: red for allergens/high risk, yellow for moderate, green for good
    - [x] 3.3.4.3 Bar chart of DV%
    - [x] 3.3.4.4 Recommendations as formatted text
    - [x] 3.3.4.5 Overall summary and risk level

  - [x] **3.3.5 Upload Page** (`src/ui/pages_upload.py`)
    - [x] 3.3.5.1 `st.file_uploader` accepting jpg/jpeg/png
    - [x] 3.3.5.2 Display uploaded image
    - [x] 3.3.5.3 Run OCR on upload, show raw text in expander
    - [x] 3.3.5.4 Show parsed data in editable `nutrition_editor`
    - [x] 3.3.5.5 "Analyze" button triggers LLM analysis

  - [x] **3.3.6 Manual Entry Page** (`src/ui/pages_manual.py`)
    - [x] 3.3.6.1 Render `nutrition_editor` with empty/zero defaults
    - [x] 3.3.6.2 Include text area for ingredients list
    - [x] 3.3.6.3 "Analyze" button triggers LLM analysis

- [x] **3.4 Food Photo Recognition (Track D — Aarav)**
  > Upload a photo of actual food (not a label) → AI identifies items + portions → pulls nutrition data from USDA → feeds into existing analysis pipeline.

  - [x] **3.4.1 Vision Prompt** (`src/llm/prompts.py`)
    - [x] 3.4.1.1 Write vision system prompt that instructs the model to identify food items, estimate portions (in grams), and return structured JSON
    - [x] 3.4.1.2 Define JSON response structure: `foods` array with `name`, `estimated_grams`, `confidence` per item
    - [x] 3.4.1.3 Include instruction to be conservative on portions and flag uncertainty

  - [x] **3.4.2 Food Identifier** (`src/vision/food_identifier.py`)
    - [x] 3.4.2.1 Write `identify_food(image_bytes) -> list[dict]` using Groq with vision model (swapped from decommissioned `llama-3.2-90b-vision-preview` → `meta-llama/llama-4-scout-17b-16e-instruct`)
    - [x] 3.4.2.2 Encode image to base64, send as image content in chat completion
    - [x] 3.4.2.3 Parse JSON response into list of identified food items
    - [x] 3.4.2.4 Add try/except with user-friendly error on failure

  - [x] **3.4.3 USDA Bridge** (`src/vision/food_identifier.py`)
    - [x] 3.4.3.1 Write `lookup_food_nutrition(food_name, grams, usda_client) -> NutritionData` — search USDA for the food, scale nutrition values to estimated portion
    - [x] 3.4.3.2 Write `aggregate_nutrition(food_items) -> NutritionData` — combine multiple foods into one `NutritionData` for analysis
    - [x] 3.4.3.3 Handle USDA miss gracefully — if food not found, flag it to the user

  - [x] **3.4.4 Snap Food Page** (`src/ui/pages_snap.py`)
    - [x] 3.4.4.1 `st.file_uploader` or `st.camera_input` for food photo
    - [x] 3.4.4.2 Display uploaded photo
    - [x] 3.4.4.3 "Identify Food" button → call vision model → show identified items + portions
    - [x] 3.4.4.4 Show editable table of identified foods (user can correct names/portions)
    - [x] 3.4.4.5 "Get Nutrition & Analyze" button → USDA lookup → populate `nutrition_editor` → LLM analysis
    - [x] 3.4.4.6 Show disclaimer: "Portions are AI-estimated — adjust if needed for accuracy"

  - [x] **3.4.5 Vision Testing**
    - [x] 3.4.5.1 Manual test: photo of simple meal (e.g. apple, sandwich) → check identified items
    - [x] 3.4.5.2 Manual test: photo of complex plate → verify reasonable portion estimates
    - [x] 3.4.5.3 Test USDA bridge with known foods → verify nutrition values are reasonable

---

### Phase 3.5 — Recipe Generator `April 9 – April 11`
> Goal: Add a 4th tab that lets users build a pantry from scanned labels or food photos, then generates a nutritious recipe. Combats food insecurity angle for the symposium.

- [x] **3.5.1 Data Models** (`src/nutrition/models.py`)
  - [x] 3.5.1.1 Add `PantryItem` dataclass — name, source, nutrition, estimated_grams, quantity
  - [x] 3.5.1.2 Add `GeneratedRecipe` dataclass — title, servings, ingredients_used, additional_ingredients_needed, instructions, estimated_nutrition, nutrition_highlights, tips

- [x] **3.5.2 Recipe Prompt Templates** (`src/llm/prompts.py`)
  - [x] 3.5.2.1 Write `build_recipe_system_prompt()` — nutritionist+chef persona, JSON output schema
  - [x] 3.5.2.2 Write `build_recipe_user_prompt(pantry_items, health_profile)` — lists ingredients + health profile

- [x] **3.5.3 Groq Recipe Generation** (`src/llm/groq_client.py`)
  - [x] 3.5.3.1 Write `GroqClient` class with `_call_with_retry()` and retry on rate limit
  - [x] 3.5.3.2 Write `generate_recipe(pantry_items, health_profile) -> GeneratedRecipe`
  - [x] 3.5.3.3 Parse JSON response into `GeneratedRecipe` with `NutritionData` for estimated nutrition

- [x] **3.5.4 Recipe Generator Tab UI** (`src/ui/pages_recipe.py`)
  - [x] 3.5.4.1 Pantry builder — label scan column + food photo column + manual add expander
  - [x] 3.5.4.2 Pantry display — item list with remove buttons + clear all
  - [x] 3.5.4.3 Recipe generation — generate/regenerate buttons, recipe display with instructions
  - [x] 3.5.4.4 Nutrition breakdown — DV% bar chart using `compute_dv_percentages`
  - [x] 3.5.4.5 Nutrition highlights, tips, and AI disclaimer

- [x] **3.5.5 Wire Up in app.py**
  - [x] 3.5.5.1 Add 4th tab "Recipe Generator" and import `render_recipe_tab()`

---

### Phase 4 — Integration `April 7 – April 9`
> Goal: Wire the three tracks together into one working app.

- [x] **4.1 Wire Up the Pipeline**
  - [x] 4.1.1 Connect upload page → OCR pipeline → editable form → LLM analysis → results display
  - [x] 4.1.2 Connect manual entry page → LLM analysis → results display
  - [x] 4.1.3 Connect snap food page → vision model → USDA lookup → editable form → LLM analysis → results display
  - [x] 4.1.4 Ensure health profile sidebar feeds into all three flows

- [x] **4.2 UX Flow Verification**
  - [x] 4.2.1 Upload a clear photo → verify OCR extracts → edit → confirm → see results
  - [x] 4.2.2 Manual entry → fill form → see results
  - [x] 4.2.3 Snap food photo → verify identified items → adjust → see results
  - [x] 4.2.4 Change health profile → re-analyze → verify recommendations change

- [x] **4.3 Error Handling**
  - [x] 4.3.1 Blurry/bad image → show warning + suggest manual entry
  - [x] 4.3.2 Groq API failure → show `st.error()` with message
  - [x] 4.3.3 USDA API failure → gracefully skip, don't crash
  - [x] 4.3.4 Missing health profile fields → still works with generic analysis

---

### Phase 5 — Evaluation `April 9 – April 12`
> Goal: Get concrete accuracy numbers for the presentation.

- [x] **5.1 LLM Evaluation** — 29/30 checks (96.7%), all four prompt dimensions at 100%, one risk-rating edge case. Raw results in `eval/llm_accuracy_results.json`.
  - [x] 5.1.1 Create 5 test cases (nutrition label + health profile combos)
  - [x] 5.1.2 Run each through the LLM pipeline
  - [x] 5.1.3 Score pass/fail: allergen detection, nutrient flagging, preservative ID, recommendation relevance
  - [x] 5.1.4 Record results for poster/presentation

---

### Phase 6 — Local Resource Finder `April 12 – April 14`
> Goal: Analyze the user's diet for nutrient gaps, then connect them to free or low-income-accessible food resources nearby. Focus exclusively on places that serve food-insecure households — no regular grocery stores or paid services.

- [ ] **6.1 Nutrient Gap Analysis**
  - [ ] 6.1.1 Compare user's scanned/entered foods against FDA daily values to identify deficiencies
  - [ ] 6.1.2 Generate a "missing nutrients" summary (e.g., "Low on iron, calcium, Vitamin D")
  - [ ] 6.1.3 Map deficiencies to food categories (leafy greens, dairy, legumes, etc.)

- [ ] **6.2 Local Resource Lookup**
  - [ ] 6.2.1 Research free APIs for low-income food access (USDA Food Desert Atlas, FoodFinder API, Feeding America locator, 211.org)
  - [ ] 6.2.2 Write `find_local_resources(zip_code, resource_type) -> list[dict]` — returns nearby free/low-cost places with name, address, hours, eligibility
  - [ ] 6.2.3 Resource types: food banks, food pantries, community fridges, free community gardens, SNAP/WIC retailers, free meal programs, subsidized farmers markets
  - [ ] 6.2.4 Filter out regular grocery stores and paid services — only free or income-qualified resources
  - [ ] 6.2.5 Fallback: curated list of West Lafayette / Lafayette free food resources (food banks, Purdue food pantry, community gardens) for video

- [ ] **6.3 LLM Recommendation Layer**
  - [ ] 6.3.1 Write prompt: given nutrient gaps + nearby free resources → personalized advice ("You're low on iron — the Lafayette Community Food Bank on Main St has free produce distributions on Saturdays")
  - [ ] 6.3.2 Frame around free/low-cost access: food bank hours, SNAP-eligible stores, free meal schedules, community garden sign-ups

- [ ] **6.4 UI — "Find Free Food Near You" Tab**
  - [ ] 6.4.1 Zip code / location input
  - [ ] 6.4.2 Display nutrient gap summary
  - [ ] 6.4.3 List of free/low-cost food resources with hours, address, and eligibility info
  - [ ] 6.4.4 Personalized LLM advice connecting nutrient gaps to specific free resources

---

### Phase 7 — Presentation + Video `April 14 – April 16`
> Goal: Research talk with recorded video walkthrough of the app. No live demo needed.

- [ ] **7.1 App Polish**
  - [ ] 7.1.1 Clean up Streamlit styling (page title, icon, colors)
  - [ ] 7.1.2 Add brief app description/instructions on main page
  - [ ] 7.1.3 Final error handling pass — no tracebacks during video recording

- [ ] **7.2 Video Walkthrough Scenarios**
  - [ ] 7.2.1 Scenario 1: Upload a clear label photo with no health concerns → basic analysis
  - [ ] 7.2.2 Scenario 2: Upload label with peanut-allergic profile → allergen flagging
  - [ ] 7.2.3 Scenario 3: Manual entry of high-sodium product with "low sodium" goal → goal mismatch
  - [ ] 7.2.4 Scenario 4: Product with preservatives → preservative warnings
  - [ ] 7.2.5 Scenario 5: Snap photo of a meal → AI identifies foods → nutrition breakdown + analysis
  - [ ] 7.2.6 Scenario 6: Recipe from scanned labels (cereal, beans, milk, bread) with "high protein" goal
  - [ ] 7.2.7 Scenario 7: Food insecurity — recipe from rice, canned beans, onion → maximize nutrition
  - [ ] 7.2.8 Scenario 8: Nutrient gap analysis → local resource recommendations
  - [ ] 7.2.9 Scenario 9: Allergen-safe recipe — peanut+dairy allergens set, verify recipe excludes them

- [ ] **7.3 Record Video**
  - [ ] 7.3.1 Run through all scenarios in the app, screen record each
  - [ ] 7.3.2 Edit into a cohesive walkthrough video (2-4 minutes)
  - [ ] 7.3.3 Add voiceover or captions explaining each feature

- [ ] **7.4 Presentation Slides**
  - [ ] 7.4.1 Problem statement — food insecurity + nutrition literacy gap
  - [ ] 7.4.2 Solution overview — NutriScan's 5 features (label scan, food snap, manual entry, recipe generator, free local resources)
  - [ ] 7.4.3 Technical architecture slide — Groq Vision (label reading + food photo identification), Groq LLM (analysis + recipe generation), USDA FoodData Central + Open Food Facts (nutrition lookups), Tesseract OCR as offline fallback
  - [ ] 7.4.4 Evaluation results (LLM checklist scores; vision-vs-OCR comparison if 7.6 is done)
  - [ ] 7.4.5 Embed or link video walkthrough
  - [ ] 7.4.6 Future work — expanded local resources, multi-language support, mobile app
  - [ ] 7.4.7 Practice talk (aim for ~10 min depending on symposium format)

- [ ] **7.5 Day-of Checklist** `April 16`
  - [ ] 7.5.1 Slides exported/uploaded and accessible
  - [ ] 7.5.2 Video plays correctly from slides
  - [ ] 7.5.3 Backup: have app running on laptop in case of Q&A ("can you show me X?")

- [ ] **7.6 Stretch — Vision vs OCR Comparison** (only if Phase 7.1-7.5 are done with time to spare)
  > Optional talking point: show the engineering pivot from Tesseract OCR to Groq vision for label reading. One slide, ~30s of talk time.
  - [ ] 7.6.1 Hand-label ground truth for the 4 photos already in `tests/sample_labels/` (fda_2014, agave_nectar, monster_energy, iphone_test) in `eval/ground_truth.json`
  - [ ] 7.6.2 Write `eval/label_reader_accuracy.py` — runs both paths (vision via `extract_label_with_vision`, OCR fallback via `extract`) on each image, prints a side-by-side field-extraction table
  - [ ] 7.6.3 Add one comparison slide to the deck: table + one-line narrative ("vision was 8-10× more accurate on real phone photos, which is why we made it the primary path")

---

## Task Division

| Person | Phase 3 | Phase 4 | Phase 5 | Phase 6 |
|--------|---------|---------|---------|---------|
| **Aarav** | ✅ 3.1.3 real-image OCR validation + regex bugfixes · ✅ 3.2 LLM Integration · ✅ 3.3.1.5 OFF fallback · ✅ 3.3.5 / 3.3.6 / 3.4.4 cleanup · ✅ 3.4 Food Photo Recognition · ✅ 3.5 Recipe Generator | ✅ 4.1 pipeline wire-up · ✅ 4.2 UX verification · ✅ 4.3 error handling · ✅ vision-based label reader | ✅ 5.1 LLM eval (29/30, 96.7%) | — |
| **Nuv** | ✅ 3.3.1 USDA scaffold · ✅ 3.3.2 Health Profile form · ✅ 3.3.3 Nutrition Editor · ✅ 3.3.4 Results Display · ✅ 3.3.5 Upload Label page scaffold · ✅ 3.3.6 Manual Entry page scaffold · ✅ 3.4.4 Snap Food page scaffold · ✅ app.py tab wiring | — | — | 6.4 Find Free Food tab UI |
| **Neil** | ✅ 3.1.1 preprocessor · ✅ 3.1.2 extractor + regex · ✅ 3.1.3.3 / 3.1.3.4 hardcoded-string tests | — | — | 6.1-6.3 Local Resources backend (gap analysis, resource lookup, LLM recs) |

### Completed Work Log (as of 2026-04-11)

**Aarav**
- Phases 1-2: scaffolding, data models, FDA daily values, DV% computation
- Phase 3.2: LLM Integration (prompts, `GroqClient.analyze()`, unit tests, manual Groq verification)
- Phase 3.3.1.5: Open Food Facts fallback client + USDA POST/X-Api-Key fix
- Phase 3.4: Food Photo Recognition — vision prompts, Groq vision client (`food_identifier.identify_food`, swapped decommissioned `llama-3.2-90b-vision-preview` → `meta-llama/llama-4-scout-17b-16e-instruct`), USDA/OFF bridge (`lookup_food_nutrition`, `aggregate_nutrition` with per-100g scaling and unit conversion), manual vision tests on 3 real meal photos
- Phase 3.3.5 / 3.3.6 / 3.4.4: fixed incorrect `lookup_food_nutrition` and `aggregate_nutrition` call signatures in `pages_snap`
- Phase 3.5: Recipe Generator feature (models, prompts, Groq client, UI, app.py wiring)
- Phase 3.1.3 real-image validation: collected 4 public-domain nutrition labels from Wikimedia Commons into `tests/sample_labels/`, ran end-to-end pipeline to surface real Tesseract artifacts, fixed 3 regex bugs — (1) `_parse_ingredients` `[A-Z]{5,}` header check was case-insensitive due to leaked `re.IGNORECASE` flag → scoped via inline `(?i:...)` on keywords only; (2) `iron` regex accepts `[il1]ron` for Tesseract's Iron→lron misread; (3) added reversed-order `_SERVINGS_PER_CONTAINER_REVERSED_RE` for FDA 2014+ "8 servings per container" layout. Added `TestRealImageExtraction` class (16 integration tests)
- Phase 4.1 pipeline wire-up: fixed broken `extract_nutrition` imports in `pages_upload.py` and `pages_recipe.py`; audited Manual Entry, Snap Food, and sidebar HealthProfile flows
- Phase 4.3 error handling: surfaced low/medium OCR confidence as `st.warning`/`st.info` banners in Upload Label and Recipe label-scan paths; verified Groq/USDA/OFF failures degrade gracefully
- Phase 4.2 UX flow verification: manually walked all four flows in browser — iPhone label upload → vision reader → editor → LLM analysis (sodium=320 matched ground truth); high-sodium manual entry flagged BHT/HFCS/65% DV sodium; Snap Food photo → vision → USDA → aggregated nutrition; sidebar health profile change → re-analysis flipped goal alignment to `low-sodium: CONFLICT`
- Vision-based label reader (`src/vision/label_reader.py`): replaced Tesseract for Upload Label + Recipe "Add from Label" flows. Groq vision system/user prompts, base64 image encoding, strict NutritionData JSON schema with unit conversion and "<1g"→0.5 rules, Tesseract fallback chain. Added `.heic`/`.heif`/`.webp` to the file_uploader allow-list plus `pillow-heif` registration so iPhone photos open natively. Preprocessor improvements kept as fallback: EXIF orientation via `ImageOps.exif_transpose`, normalize width to 1600px, adaptive threshold block=25/C=2. Extractor cleanup heuristics (letter-to-digit OCR fixups, loose `incl\w*` / `[il1]ron` / `[m\s]*g?` patterns, optional trailing g, multi-group alternation for modern "Includes Xg Added Sugars") as safety net for the fallback path
- 80/80 tests passing across the Phase 4 work
- Phase 5.1 LLM Evaluation: 5 realistic test cases in `eval/llm_test_cases.py` covering clean-pass, allergen detection, preservative flagging, goal conflict, and multi-issue cascade. Runner + scorer in `eval/llm_accuracy.py` uses case-insensitive substring matching against AnalysisResult fields (concept capture, not exact wording), regenerates `eval/llm_accuracy_report.md` and optional JSON dump on every run. Results: 29/30 checks passed (96.7%) — all four core prompt dimensions (allergen, preservative, nutrient, goal) scored 100%; single miss was the LLM rating a preservative-heavy 1-oz chip serving as `low` risk vs expected `moderate`/`high` (defensible per-serving interpretation, test case left unchanged to keep the number honest)

**Nuv**
- Phase 3.3.1: `search_food` + `check_preservatives`
- Phase 3.3.2: Health Profile sidebar form
- Phase 3.3.3: Nutrition Editor widget (shared across Upload / Manual / Snap / Recipe tabs)
- Phase 3.3.4: Results Display page (colored flags, DV% bar chart, recommendations, risk summary)
- Phase 3.3.5: Upload Label page scaffold
- Phase 3.3.6: Manual Entry page scaffold
- Phase 3.4.4: Snap Food page scaffold (camera + upload, editable food table, pipeline wiring)
- Wired up all five tabs in `app.py`

**Neil**
- Phase 3.1.1 preprocessor (`src/ocr/preprocessor.py`): PIL/path/numpy loader, grayscale, upscale, adaptive threshold, Gaussian blur
- Phase 3.1.2 extractor (`src/ocr/extractor.py`): Tesseract invocation with `--psm 6`, per-nutrient regex patterns, ingredients parser, confidence indicator
- Phase 3.1.3.3 / 3.1.3.4: hardcoded-string unit tests in `tests/test_ocr.py` covering clean / spaced / decimal / sparse / noisy label formats

---

## Verification Summary
- **Unit tests:** OCR parsing, DV% math, prompt construction, response parsing, recipe generation (`pytest tests/`)
- **Integration tests (manual):** Clear photo, blurry photo, food snap, allergen scenario, diet goal scenario, empty profile, recipe from pantry, free resource lookup
- **Evaluation:** LLM checklist on 5 test cases (Phase 5.1); vision food ID spot checks; recipe quality spot checks; optional vision-vs-OCR comparison as a stretch slide (Phase 7.6)
- **Video recording:** Run all 9 walkthrough scenarios (Phase 7.2) and screen record before April 16
- **Pre-talk check:** Slides + video ready, app running as backup for Q&A

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AaravChadha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
