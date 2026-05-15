## thematic-research-summary-instructions-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

*This project is a fork of the original Survey Scoring Prompt Generator, repurposed to focus on summarization instructions rather than classification scoring.*

This project is an interface for generating and iterating prompts that provide instructions for creating comprehensive summaries of survey responses. The goal is to create a user-friendly system that helps users craft and refine prompts used by LLMs to summarize survey data effectively.

## Project Structure

```
summary_prompt_gen_ux/
├── reference_materials/
│   ├── llm_call_example.py      # Empty placeholder file  
│   └── llm_claude_example.py    # AWS Bedrock + Claude integration reference
```

## Core Functionality

The main purpose is to build an interface that allows users to:
- Generate prompts for survey response summarization
- **AI-powered summary type generation** - Automatically create relevant summary categories based on summarization criteria
- **CSV data import** - Upload and process custom survey data with conversation grouping
- Iterate and refine those prompts based on results
- Test prompts against sample survey data or uploaded CSV files
- **Export final prompts** - Copy refined instructions for external LLM integration
- Optimize prompt effectiveness for consistent, comprehensive summarization

## LLM Integration Reference

The `reference_materials/llm_claude_example.py` shows how to integrate with Claude via AWS Bedrock:
- AWS Bedrock runtime client setup for `us-east-1` region
- Claude 3.7 Sonnet inference profile usage
- Proper API request structure with anthropic_version and message format
- Response handling and text extraction

## Dependencies

Current reference dependencies:
- `boto3` for AWS Bedrock integration
- `pandas` for data handling
- Standard Python libraries (`math`, `json`, `random`, `textwrap`)

## Development Commands

### Setup
```bash
# Create virtual environment (first time only)
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Running the Application
```bash
# Option 1: Use startup script (recommended)
./run.sh

# Option 2: Use startup script with dev mode (shows step 4)
./run.sh --dev

# Option 3: Manual startup
source venv/bin/activate
python3 app_frontend.py
```

The application will be available at http://localhost:5000

### Running Tests or Python Scripts
**IMPORTANT: Always activate the virtual environment before running any Python scripts**
```bash
# ALWAYS activate virtual environment first
source venv/bin/activate

# Then run Python scripts
python3 test_unseen_selection.py
python3 any_script.py
```

**Note:** If port 5000 is in use (common on macOS due to AirPlay), the run.sh script will automatically kill conflicting processes. For manual startup, use a different port if needed.

### Development Status

- Flask web application with 8-step workflow implemented
- **Smart summary type generation** - Uses LLM to generate contextually relevant summary categories
- **CSV upload functionality** - Import custom survey data from CSV files
- **Export instructions feature** - Export final prompts for external use
- Virtual environment setup with all dependencies
- Multiple sample survey datasets included
- End-to-end functionality complete with user feedback integration

## 8-Step Workflow Overview

The application guides users through a systematic 8-step process to create, test, and refine prompts for summarizing survey responses.

**Linear Workflow:**
1. **Step 1:** Select Survey Example → Choose from available datasets or upload CSV
2. **Step 2:** Summarization Description → Define summarization criteria  
3. **Step 3:** Generate Summary Types → AI-powered summary categories
4. **Step 4:** Review Prompt → AI-generated expert summarization prompt
5. **Step 5:** Run Inference → Batch process all responses
6. **Step 6:** Provide Feedback → Review and refine AI summaries
7. **Step 7:** Final Results → View completed summarization results
8. **Step Final:** Export Instructions → Copy final prompt for external use

**Key Innovation:** The system uses AI at multiple stages to create contextually relevant summary types and expert-level prompts, then processes all responses in a single efficient batch call rather than individual API requests.

## Detailed Step Explanations

### Step 1: Select Survey Example or Upload CSV
**Purpose:** Choose from pre-loaded survey datasets or upload custom CSV data

**User Experience:** 
- **Tab 1: Built-in Examples** - View available survey datasets with titles and response counts
- **Tab 2: Upload CSV** - Upload custom survey data from CSV files
- Built-in examples include: Customer Satisfaction, Employee Feedback, Product Reviews
- CSV upload supports drag-and-drop or click-to-browse functionality

**Technical Implementation:**
- **Built-in Examples:** Loads JSON files from `examples/` directory using `load_survey_examples()`
- **CSV Upload:** Processes CSV files with automatic delimiter detection and column selection
- Selected data stored in `session['survey_data']` for subsequent steps
- Supports up to 50 responses per dataset (limit applied in Step 5)

**CSV Upload Process:**
1. **File Upload:** Drag-and-drop or browse for CSV file
2. **Column Selection:** Choose required columns:
   - **Text Column:** Contains the survey response text (required)
   - **Conversation ID Column:** Groups messages into conversations (required)
   - **Author Column:** Identifies message authors (Agent/Customer) (required)
3. **Data Transformation:** CSV data converted to conversation format matching internal structure
4. **Preview:** Truncated values (50 chars max) shown for column selection

**Input Formats:** 
- **JSON files** with structure:
```json
{
  "title": "Survey Name",
  "description": "Survey description", 
  "responses": [
    {"id": 1, "text": "Survey response text"},
    {"id": 2, "text": "Another response"}
  ]
}
```

- **CSV files** with columns for:
  - Response text (any column name)
  - Conversation ID (for grouping related messages)
  - Author/Speaker identification (Agent, Customer, etc.)

---

### Step 2: Summarization Description  
**Purpose:** Define what aspects you want to summarize in the survey responses

**User Experience:**
- Enter free-text description of summarization criteria 
- Suggestion: 2-3 sentences work well (not enforced)
- Example: "I want to create summaries focusing on customer satisfaction themes and product quality feedback"

**Technical Implementation:**
- User input stored in `session['summarization_description']`
- Triggers AI-powered summary type generation via `generate_smart_summary_types()`
- Handles empty/invalid input with flash messages
- No character limits or strict formatting requirements

**AI Integration:** This description becomes the foundation for contextual summary type generation in Step 3

---

### Step 3: Generate Summary Types (AI-Enhanced)
**Purpose:** Create summary type categories tailored to your specific summarization criteria

**User Experience:**
- Review AI-generated summary type categories 
- Edit summary type names and descriptions as needed
- Multiple summary types generated: key_themes, sentiment_overview, specific_issues, general_feedback
- Example for "customer experience": "Key Experience Themes", "Overall Sentiment", "Specific Pain Points", "General Comments"

**Technical Implementation:**
- Uses `generate_smart_summary_types(summarization_description)` to create contextual categories
- Makes LLM call with detailed prompt requesting multiple summary type categories
- Parses JSON response for summary type names and descriptions
- Fallback to `generate_default_summary_types()` if AI generation fails
- Final summary types stored in `session['summary_types']` as nested dictionary

**AI Prompt Strategy:**
- Requests summary types specifically tailored to the summarization criteria
- Ensures coverage of different summary perspectives and focus areas
- Returns structured JSON with names and detailed descriptions

**Inference Model Selection (NEW):**
- Dropdown to choose the model for Step 5 inference processing
- **Claude 3 Haiku** (default): Fast & cost-effective, ideal for bulk processing
- **Claude 3.7 Sonnet**: Advanced reasoning & quality when accuracy is critical
- Model selection only affects inference; all other operations use Sonnet
- Selected model stored in session and displayed in Step 6 results

---

### Step 4: Review Prompt (AI-Enhanced)
**Purpose:** Generate and refine the expert summarization prompt for consistent summaries

**User Experience:**
- Review comprehensive AI-generated summarization prompt
- Edit prompt text as needed in large textarea
- Prompt includes summarization criteria, summary type definitions, and consistency guidelines

**Technical Implementation:**
- Uses `generate_initial_prompt(summarization_description, summary_types)` for expert prompt creation
- Makes LLM call requesting professional prompt with batch processing instructions
- Designed specifically for batch inference with JSON output format
- Fallback to `generate_template_prompt()` if AI generation fails
- Final prompt stored in `session['initial_prompt']`

**Prompt Engineering:** 
- Creates expert-level instructions for consistent summarization
- Includes edge case handling and ambiguity resolution guidance
- Optimized for batch processing of multiple responses simultaneously
- Does NOT include output format instructions (added separately in Step 5)

---

### Step 5: Run Inference (Batch Processing)
**Purpose:** Process all survey responses using the finalized prompt to generate summaries with maximum efficiency

**User Experience:**
- Click "Start Processing" to begin batch inference
- Watch progress bar with realistic timing simulation
- Automatic redirect to Step 6 when complete
- Processing message: "Batch processing all responses... X%"

**Technical Implementation:**
- Extracts up to 50 response texts from survey data (configurable via `INFERENCE_LIMIT`)
- **Dynamic progress display:** Shows actual count of conversations being processed
- **Model Selection:** Uses the inference model selected in Step 3 (Haiku or Sonnet)
- Uses `run_individual_inference()` for better summary quality with selected model
- Makes individual API calls per response with the chosen model
- Stores results in `session['inference_results']` with dual field structure
- Exports results to timestamped JSON file in `temp_results/` directory
- Tracks which model was used for later display in Step 6

**Progress Display Features:**
- **Real-time counting:** Progress bar denominator matches actual conversation count
- **Configurable limits:** `INFERENCE_LIMIT = 3` (development) adjustable for production
- **Accurate feedback:** No hardcoded values - dynamically calculated from dataset size

**Batch Processing Benefits:**
- **50x faster** than individual API calls
- **50x cheaper** in API costs
- **Better consistency** across summaries
- **Robust error handling** with detailed debug logging

**Data Structure:** Results stored with both old and new field names for compatibility:
```python
{
  'response_text': text,        # New format
  'response': text,             # Backward compatibility  
  'ai_summary': summary,        # New format
  'summary': summary,           # Backward compatibility
  'index': i,
  'full_response': batch_response
}
```

---

### Step 6: Provide Feedback (Interactive)
**Purpose:** Review AI-generated summaries and provide corrections to improve prompt effectiveness

**User Experience:**
- **Model Display:** Shows which model (Haiku or Sonnet) was used for inference
- View all survey responses with AI-generated summaries
- See up to 10 examples per summary type category (organized view)
- Emergency fallback shows all responses if summary matching fails
- Edit summaries via text areas or dropdown menus
- Optionally add feedback text explaining changes
- Visual indicators show modified responses
- Submit feedback with change summary confirmation
- **Unlimited Iterations:** Continue refining prompts as many times as needed

**Technical Implementation:**
- Displays `session['inference_results']` with comprehensive debug information
- Uses fuzzy matching logic to organize responses by summary type
- Tracks changes between original and corrected summaries
- JavaScript handles interactive change indicators and progress tracking
- Collects feedback data via AJAX call to `/submit_feedback`

**Visual Features:**
- **AI Result Section:** Blue background showing AI summary
- **User Correction Section:** Yellow background for user changes  
- **Change Indicators:** Visual markers for modified responses
- **Debug Panel:** Shows summary counts and matching statistics
- **Fallback Display:** Ensures all responses visible even if categorization fails

**Data Collection:** Gathers detailed feedback including:
- Original vs corrected summaries
- Optional explanatory text
- Change counts and summary statistics

---

### Step 7: Final Results
**Purpose:** View completed summarization results with user feedback incorporated

**User Experience:** 
- See summary results with counts per summary type
- Final results incorporating any user corrections from Step 6
- Option to start new session
- Classification statistics display

**Technical Implementation:**
- Displays `session['final_results']` with aggregated statistics
- Calculates summary type counts using dictionary aggregation
- Provides clean interface for starting new workflow
- Maintains session data for reference

**Workflow Completion:** 
- Displays final results immediately after Step 6 feedback submission
- Incorporates any user corrections made during feedback phase
- Provides summary statistics and option to begin new summarization session

---

### Step Final: Export Instructions
**Purpose:** Export the final summarization prompt for use in external applications

**User Experience:**
- Click "I'm Happy - Export Instructions" button from Step 6
- View final prompt with proper newline formatting preserved
- Copy instructions as JSON string with single button click
- Visual confirmation when copy operation succeeds
- Option to return to Step 6 or start new session

**Technical Implementation:**
- Accessible via `/step/final` route (Step 8 in internal numbering)
- Retrieves final prompt from `session['initial_prompt']`
- Displays instructions with `<pre>` tags for formatting preservation
- JavaScript handles clipboard operations with modern API + fallback
- JSON export format: `{"instructions": "prompt_text"}`

**Export Features:**
- **Cross-browser compatibility:** Modern `navigator.clipboard` with `document.execCommand` fallback
- **Proper formatting:** Preserves all newlines and spacing from original prompt
- **JSON structure:** Ready for integration with external LLM applications
- **User feedback:** Success message with auto-fade after 3 seconds
- **Navigation options:** Return to feedback step or start fresh workflow

**Use Cases:**
- Integration with custom LLM applications
- Sharing refined prompts across teams
- Documentation of prompt engineering iterations
- External API integration with consistent prompt structure

## Technical Implementation Details

### Session Management
- **Dual-layer architecture:** Flask sessions + backend consolidated session data
- **Backend consolidation:** Uses `SessionManager` for persistent data storage in `temp_results/`
- **Session files:** Individual JSON files per session ID (e.g., `session_ABC123_object.json`)
- Key session variables: `step`, `selected_example`, `survey_data`, `summary_description`, `summary_instructions`, `classes`, `initial_prompt`, `inference_results`, `user_feedback`, `final_results`
- Session cleared on new workflow start with new UUID session ID
- Persistent across browser refresh and application restarts

### Data Flow Architecture
1. **Step 1:** Load examples OR CSV upload → session['survey_data'] + backend consolidation
2. **Step 2:** User input → session['summary_description'] → AI summary type generation
3. **Step 3:** AI summary types → user editing → session['summary_instructions'] → AI prompt generation  
4. **Step 4:** AI prompt → user editing → session['initial_prompt'] + backend consolidation
5. **Step 5:** Batch processing → session['inference_results'] + temp file export + backend consolidation
6. **Step 6:** User feedback → session['user_feedback'] → session['final_results'] + backend consolidation
7. **Step 7:** Final display with user corrections incorporated
8. **Step Final:** Export final prompt as JSON string for external use

### Backend Architecture
- **Framework separation:** `app_frontend.py` (Flask UI) + `app_backend.py` (business logic)
- **Session consolidation:** `get_consolidated_session_data()` and `save_consolidated_session_data()`
- **Cross-framework compatibility:** Backend designed to work with Flask, terminal, or other interfaces
- **File-based persistence:** Session data survives application restarts

### Error Handling & Fallbacks
- **AI Generation Failures:** Automatic fallback to template-based approaches
- **API Call Failures:** Error results created for all responses with diagnostic info
- **JSON Parsing Errors:** Detailed logging with response preview for debugging  
- **Session Loss:** Debug panels show session state for troubleshooting
- **Summary Mismatches:** Fuzzy matching + emergency fallback display

### File Structure
```
summary_prompt_gen_ux/
├── app_frontend.py           # Flask web interface (main entry point)
├── app_backend.py            # Core business logic (framework-agnostic)
├── app_terminal.py           # Terminal interface
├── run.sh                   # Startup script with dev mode support
├── templates/
│   ├── index.html           # Single-page application template
│   ├── components/          # Reusable template components
│   └── steps/               # Step-specific templates
├── static/
│   ├── style.css            # Professional styling
│   ├── app.js               # Interactive JavaScript
│   ├── css/                 # Additional stylesheets
│   └── js/                  # Additional JavaScript modules
├── examples/                 # Sample survey datasets
│   ├── conversation_list_sample100.json # Full conversation examples
│   └── product_reviews.json # Product review dataset
├── uploads/                 # Temporary CSV upload storage
├── temp_results/            # Session data and inference results
├── reference_materials/     # LLM integration examples
│   ├── llm_call_example.py
│   ├── llm_claude_example.py
│   └── prompting_examples.py
├── venv/                    # Virtual environment
├── requirements.txt         # Python dependencies
└── README.md               # Project documentation
```

## Key Features Summary

- **AI-Powered Summary Type Generation:** Context-specific summary categories instead of generic templates
- **Expert Prompt Creation:** Professional prompts with consistency guidelines  
- **Flexible Model Selection:** Choose between Claude 3 Haiku (fast/economical) or Claude 3.7 Sonnet (quality) for inference
- **CSV Upload Support:** Import custom survey data with conversation grouping
- **Dynamic Progress Display:** Real-time progress tracking with accurate conversation counts
- **Export Instructions Feature:** Copy final prompts as JSON for external use
- **Interactive Feedback Interface:** Visual distinction between AI vs human corrections
- **Unlimited Iterations:** No artificial limits - refine prompts as many times as needed
- **Robust Session Management:** Dual-layer persistence with backend consolidation
- **Cross-browser Compatibility:** Modern clipboard API with fallback support
- **Robust Error Handling:** Comprehensive fallbacks and debug information
- **Results Export:** Timestamped JSON files for analysis and debugging
- **Professional UI/UX:** Progress tracking, change indicators, and intuitive workflow

## Usage Examples

### Complete Workflow Example

**Scenario:** Summarizing customer satisfaction survey responses

1. **Step 1:** Select "Customer Satisfaction Survey" (15 responses) OR upload CSV file with conversation data
2. **Step 2:** Enter "I want to create summaries focusing on key satisfaction themes, sentiment patterns, and specific product feedback"  
3. **Step 3:** Review AI-generated summary types and **select inference model**:
   - AI generates: "Key Satisfaction Themes", "Overall Sentiment Patterns", "Specific Product Feedback", "General Comments"
   - Choose **Claude 3 Haiku** (fast/economical) or **Claude 3.7 Sonnet** (quality) for processing
4. **Step 4:** Review and edit comprehensive AI-generated summarization prompt (15+ lines)
5. **Step 5:** Process all 15 responses using selected model with dynamic progress tracking
6. **Step 6:** Review results (shows which model was used), refine summaries, provide unlimited feedback
7. **Step 7:** View final results: 4 comprehensive summary categories with key insights extracted
8. **Step Final:** Click "I'm Happy - Export Instructions" → Copy final prompt as JSON string for external use

### Expected Performance

**With Claude 3 Haiku (Default):**
- **Processing Time:** 2-3 seconds per response (faster than Sonnet)
- **API Cost:** ~60-80% cheaper than Claude 3.7 Sonnet
- **Quality:** Good for most summarization tasks
- **Best for:** Bulk processing, cost optimization

**With Claude 3.7 Sonnet (Optional):**
- **Processing Time:** 3-5 seconds per response (higher quality takes time)
- **API Cost:** Higher but provides premium quality
- **Quality:** Superior reasoning and nuanced understanding
- **Best for:** Complex summaries requiring detailed analysis

**Model Selection Benefits:**
- **Cost Optimization:** Use Haiku for routine processing, Sonnet when quality is critical
- **Flexibility:** Choose the right tool for each task
- **Unlimited Iterations:** Refine prompts as many times as needed to perfect results

## AWS Configuration

For Bedrock integration, the application uses a dual-model approach:

### Primary Model (for prompt generation, refinement)
- **Model:** Claude 3.7 Sonnet
- **ARN:** `arn:aws:bedrock:us-east-1:457209544455:inference-profile/us.anthropic.claude-3-7-sonnet-20250219-v1:0`
- **Usage:** Summary type generation, prompt creation, prompt refinement
- **Max tokens:** 2000

### Inference Model Options (for Step 5 processing)
1. **Claude 3 Haiku (Default)**
   - **ARN:** `arn:aws:bedrock:us-east-1:457209544455:inference-profile/us.anthropic.claude-3-haiku-20240307-v1:0`
   - **Usage:** Fast, cost-effective inference processing
   - **Max tokens:** 2000
   
2. **Claude 3.7 Sonnet (Optional)**
   - **ARN:** Same as primary model
   - **Usage:** When higher quality summaries are needed
   - **Max tokens:** 2000

### Configuration
- **Region:** `us-east-1`
- **Client:** AWS Bedrock Runtime
- **Authentication:** AWS credentials (environment variables or IAM role)

---
> Source: [rd2345/thematic-research-summary-instructions-generator](https://github.com/rd2345/thematic-research-summary-instructions-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
