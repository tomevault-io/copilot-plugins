## getrelevantpapers

> - **Default Port**: 3444

# Cursor Rules for Paper Analysis Pipeline

## Web UI Configuration
- **Default Port**: 3444
- **Web UI Command**: `python src/ui/minimal_web_ui.py`
- **Access URL**: http://localhost:3444

## Project Structure
- Main pipeline: `main.py`
- Web UI: `src/ui/minimal_web_ui.py` (PORT 3444)
- Analysis: `src/core/analyzer.py` (regex) + `src/evaluators/bedrock_evaluator.py` (Bedrock LLM)
- Classifiers: `src/classifiers/bedrock_classifier.py` (for colleague's workflow)
- Data: `data/markdown_db/` + `data/vector_store/` + `data/converted_papers/`

## Development Commands
```bash
# Start Web UI on port 3444
python src/ui/minimal_web_ui.py

# CLI Analysis
python main.py --input-dir data/converted_papers

# Colleague's Workflow (Search & Classify)
python asta.py                    # Search papers
python classify_papers.py         # Classify with Bedrock
python download_papers.py         # Download PDFs
python visualize_papers.py        # Visualize results
```

## Key Features
- 22 converted sample papers in `data/converted_papers/`
- 5 Deep Researcher criteria evaluation
- Bedrock-based analysis (parallel processing)
- Interactive web interface on port 3444
- Vector search for section relevance
- Persistent document database

## API Endpoints (Port 3444)
- GET `/` - Dashboard
- GET `/papers` - Papers list
- GET `/results` - Analysis results
- POST `/analyze` - Run analysis
- GET `/api/stats` - System stats
- GET `/api/analyze/{method}/{doc_id}` - Single paper analysis

## Always use port 3444 for web UI development and testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jan21) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
