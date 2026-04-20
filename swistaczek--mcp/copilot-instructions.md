## mcp

> Generates accessible descriptions for images and GIFs using Gemini LLM.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal MCP servers collection built with FastMCP framework. The project uses Python 3.13+, mise for task management, and uv for dependency management.

## Development Commands

```bash
# Setup and Dependencies
mise run install              # Install project dependencies via uv

# Testing
uv run pytest tests/ -v       # Run full test suite (73 passed, 14 skipped expected)
uv run pytest tests/test_domains.py::TestTLDExtraction -v  # Run specific test class
uv run pytest tests/ -k "chinese" -v  # Run tests matching pattern
uv run pytest tests/ -m "not integration" -v  # Skip integration tests

# Development
mise tasks                    # List all available mise tasks
mise run dev                  # Run server in development mode
```

## Architecture

### MCP Server Pattern
Each server follows the FastMCP pattern:
1. Import FastMCP and create instance: `mcp = FastMCP("Server Name")`
2. Define tools using `@mcp.tool` decorator with Pydantic field validation
3. Use async functions for I/O operations (WHOIS queries, DNS lookups)
4. Return `ToolResult` with both human-readable text and structured JSON

### Domain Checker Architecture
Multi-layer domain availability verification with optional OVH browser-based validation.

**Verification Layers:**
1. **Primary**: WHOIS protocol queries (port 43, 10s timeout)
2. **Fallback**: DNS record lookup when WHOIS fails (potential false positives)
3. **Optional OVH Verification**: Browser automation to catch false positives

**Key Features:**
- **Batch processing**: Up to 50 domains per request
- **TLD support**: 40+ TLDs including compound (.com.cn) and Chinese IDN domains
- **Pattern matching**: TLD-specific "not found" patterns in `NOT_FOUND_PATTERNS` dict
- **False positive detection**: OVH verification catches DNS-based false positives
- **Graceful degradation**: OVH verification failures don't break core functionality

**Tool:**
`check_domains(domains, verify_available?)` - Check domain registration status
- `domains: list[str]` - Domain names to check (1-50)
- `verify_available: bool = False` - Enable OVH verification for available results

**OVH Verification Module** (`ovh_verifier.py`):
- Browser-based domain availability check using OVH's web interface
- Bypasses OVH API bot protection via Playwright automation
- Provides pricing information for available domains
- Catches false positives: domains reported available by DNS but actually registered
- **Detects aftermarket/premium domains**: Distinguishes between standard registration and secondary market

**Aftermarket Detection:**
OVH displays domain type labels that indicate aftermarket/resale domains:
- **"Premium"** - Premium domains sold at higher prices
- **"Sprzedaż przez stronę trzecią"** - Third-party sales (external marketplace)

The verifier parses these labels and marks such domains as NOT available for standard registration.

**Verification Flow:**
```python
1. WHOIS query for each domain
2. If WHOIS fails → DNS fallback (⚠️ potential false positives)
3. If verify_available=True and domain appears available:
   → OVH browser verification checks availability
   → Detects aftermarket labels (Premium, third-party)
   → Aftermarket domains marked as NOT available
   → False positives flagged if OVH says registered
```

**Response Structure (with OVH and Aftermarket):**
```json
{
  "results": {
    "catch.dev": {
      "registered": false,
      "available": false,
      "method": "dns",
      "reason": "AFTERMARKET: Domain on secondary market (Premium) - 1 384,70 zł",
      "aftermarket": true,
      "aftermarket_type": "Premium",
      "aftermarket_price": "1 384,70 zł",
      "ovh_verification": {
        "ovh_available": false,
        "ovh_verified": true,
        "ovh_price": "1 384,70 zł",
        "ovh_price_type": "premium",
        "ovh_is_aftermarket": true,
        "ovh_aftermarket_type": "Premium",
        "ovh_error": null
      }
    },
    "erne.dev": {
      "registered": false,
      "available": true,
      "method": "dns",
      "reason": "OVH CONFIRMED available (23,39 zł)",
      "ovh_confirmed": true,
      "standard_price": "23,39 zł",
      "ovh_verification": {
        "ovh_available": true,
        "ovh_verified": true,
        "ovh_price": "23,39 zł",
        "ovh_price_type": "standard",
        "ovh_is_aftermarket": false,
        "ovh_aftermarket_type": null,
        "ovh_error": null
      }
    }
  },
  "available_domains": ["erne.dev"],
  "aftermarket_domains": ["catch.dev"],
  "registered_domains": [],
  "failed_domains": [],
  "ovh_verification": {
    "enabled": true,
    "confirmed_available": ["erne.dev"],
    "aftermarket": ["catch.dev"],
    "false_positives": [],
    "verification_failed": [],
    "duration_seconds": 18.36
  }
}
```

**Domain Categories:**
- `available_domains` - Standard registration at normal prices
- `aftermarket_domains` - Secondary market (Premium/third-party) with high prices
- `registered_domains` - Already registered, not for sale
- `failed_domains` - Check failed (unsupported TLD, network error)

**Dependencies:**
- `playwright` - Browser automation (optional, for OVH verification)
- `socket` - DNS resolution
- `asyncio` - Async WHOIS queries

**Known Limitations:**
- DNS fallback has high false positive rate for .dev/.app TLDs (domains can be registered without DNS records)
- OVH verification is slower (~2-3s per domain) but more accurate
- OVH verification requires Playwright installation: `pip install playwright && python -m playwright install chromium`
- Aftermarket prices can be 10-100x standard registration prices

### Tablica Rejestracyjna PL Architecture
Integration with Polish license plate reporting website (tablica-rejestracyjna.pl) for traffic violation reporting.

**Key Features:**
- **Fetch Comments**: Retrieve all comments/reports for a license plate
- **Submit Complaints**: Post new complaints with images and descriptions
- **Image Processing**: Automatic HEIC to JPEG conversion and downscaling
- **LLM Integration**: Tool prompts LLM to analyze images first, then generate Polish descriptions

**Tools:**
1. `fetch_comments(plate_number)` - Get existing reports for a plate
   - Parses HTML to extract comment text, ratings, timestamps
   - Returns structured data with all comments
   - Validates Polish plate format (e.g., WW12345, KR1234)

2. `submit_complaint(plate_number, violation_description, image_path, location?)` - Submit new report
   - **LLM Workflow**: Tool instructs calling LLM to:
     1. Analyze the violation image from pedestrian perspective
     2. Generate contextual Polish description
     3. Pass description to the tool
   - Converts HEIC images to JPEG automatically (requires pillow-heif)
   - Downscales large images (max 1920px, 85% quality)
   - Submits via multipart/form-data POST request
   - Anonymous submission (no authentication required)

**Image Processing Pipeline:**
```python
1. Detect format (HEIC, PNG, JPG, etc.)
2. Load with PIL (pillow-heif for HEIC)
3. Downscale if width or height > 1920px (maintains aspect ratio)
4. Convert to RGB if needed (RGBA → white background)
5. Compress as JPEG (quality 85)
6. Return bytes for upload
```

**Dependencies:**
- `aiohttp` - Async HTTP requests
- `beautifulsoup4` + `lxml` - HTML parsing
- `pillow-heif` - HEIC format support (optional but recommended)
- `Pillow` - Image processing
- `aioresponses` - HTTP mocking for async tests

**Usage Example:**
```python
# LLM workflow for submitting a complaint:
# 1. User provides: plate="WW12345", image="/path/to/violation.heic"
# 2. LLM analyzes image and generates Polish description
# 3. LLM calls: submit_complaint(
#      plate_number="WW12345",
#      violation_description="Samochód zaparkowany na chodniku...",
#      image_path="/path/to/violation.heic"
#    )
```

### EXIF Metadata Extractor Architecture
Extracts EXIF metadata from images (PNG, JPEG, HEIC) with focus on GPS data and automatic reverse geocoding to street addresses.

**Key Features:**
- **Universal Format Support**: PNG, JPEG, HEIC (via pillow-heif)
- **GPS Extraction**: Coordinates, altitude, speed, direction, timestamp, accuracy
- **Reverse Geocoding**: Automatic address lookup via Nominatim (OpenStreetMap)
- **Batch Processing**: Up to 50 images per request with progress reporting
- **No API Key Required**: Uses free Nominatim service

**Tool:**
`analyze_image_metadata(images, include_address, zoom_level, batch_size)` - Extract EXIF and geocode GPS coordinates
- `images: list[str]` - Image paths (1-50)
- `include_address: bool = True` - Enable reverse geocoding
- `zoom_level: int = 18` - Detail level (18=street, 16=area, 14=city)
- `batch_size: int = 10` - Concurrent processing limit

**Prioritized EXIF Fields:**
- **GPS Data**: Latitude/longitude (decimal degrees), altitude (meters), speed (km/h), direction (degrees), timestamp (UTC), accuracy (meters)
- **Timestamps**: DateTime, DateTimeOriginal, DateTimeDigitized
- **Camera Info**: Make, Model, Software, LensModel (when available)
- **Image Metadata**: Format, size, mode, orientation, resolution

**GPS Parsing:**
```python
# Extract GPS IFD (tag 0x8825) from EXIF
# Convert DMS (degrees/minutes/seconds) to decimal degrees
# Apply hemisphere references (N/S for latitude, E/W for longitude)
# Parse additional metadata (altitude, speed, direction, timestamp)
```

**Reverse Geocoding:**
- Service: Nominatim (OpenStreetMap API)
- Rate limit: 1 request/second (automatically enforced)
- Returns: Street, city, state, postal code, country, display name
- Handles failures gracefully (returns metadata without address)

**Output Structure:**
```json
{
  "image_path": {
    "format": "HEIF",
    "size": [4032, 3024],
    "exif": {
      "make": "Apple",
      "model": "iPhone 13 Pro",
      "datetime": "2025-07-27 09:55:45"
    },
    "gps": {
      "latitude": 52.408447,
      "longitude": 16.867817,
      "altitude_meters": 88.46,
      "speed_kmh": 0.18,
      "direction_degrees": 196.24,
      "timestamp_utc": "2025-07-27T07:55:44+00:00",
      "accuracy_meters": 3.54
    },
    "address": {
      "road": "Konstancji Łubieńskiej",
      "city": "Poznań",
      "country": "Polska",
      "postcode": "60-378"
    }
  }
}
```

**Dependencies:**
- `Pillow` - Image loading and EXIF extraction
- `pillow-heif` - HEIC format support
- `aiohttp` - Async HTTP for geocoding API

**Use Cases:**
- Photo geolocation analysis
- Travel photo mapping
- Privacy auditing (check for GPS data before sharing)
- Photo organization by location
- Forensic metadata extraction

### Plate Recognition Architecture
Analyzes traffic violation photos using Gemini's vision API to extract license plates and identify which vehicle is most likely committing a violation.

**Key Features:**
- **Plate Extraction**: Identifies all visible license plates in an image
- **Violation Detection**: Determines which vehicle is committing a traffic violation
- **Contextual Reasoning**: Provides one-sentence explanation from pedestrian perspective
- **Multi-Vehicle Support**: Handles images with multiple cars
- **Simple Single-Tool Design**: One tool for complete analysis

**Tool:**
`recognize_plates(image_path, model?)` - Analyze traffic photo for plates and violations
- `image_path: str` - Path to the violation photo
- `model: str = "gemini-flash-latest"` - Optional Gemini model override

**Violation Detection Focus:**
- Parking on sidewalks
- Blocking pedestrian crosswalks
- Illegal parking in restricted zones
- Blocking pedestrian access
- Other pedestrian-affecting violations

**Image Processing:**
```python
1. Load image from file path
2. Optimize for Gemini API (max 1920px, JPEG quality 85)
3. Convert RGBA/LA/L/P modes to RGB with white background
4. Send to Gemini with specialized prompt
5. Parse JSON response with plate numbers and reasoning
```

**Output Structure:**
```json
{
  "plates": ["WW12345", "KR1234"],
  "violation_vehicle": "WW12345",
  "reasoning": "Vehicle WW12345 is parked on the sidewalk, blocking pedestrian access.",
  "metadata": {
    "image_path": "/path/to/photo.jpg",
    "model": "gemini-flash-latest",
    "plates_count": 2
  }
}
```

**Gemini Prompt Strategy:**
- Analyze from pedestrian perspective
- Focus on violations affecting pedestrians
- Extract plates as they appear (with spaces/dashes)
- Return structured JSON with plates array, violation_vehicle, and reasoning
- Handle cases with no plates or no violations

**Dependencies:**
- `google-generativeai` - Gemini API client
- `Pillow` - Image loading and optimization
- `fastmcp` - FastMCP framework

**Use Cases:**
- Automated traffic violation reporting
- Batch analysis of violation photos
- Integration with complaint submission systems (e.g., Tablica MCP)
- Pedestrian advocacy and documentation

**Integration with Tablica MCP:**
Can be used together with Tablica MCP server for complete workflow:
1. Use `recognize_plates` to analyze photo and identify violating vehicle
2. Use `submit_complaint` from Tablica MCP to report the violation

### Image Descriptions Architecture
Generates accessible descriptions for images and GIFs using Gemini LLM.

**Key Features:**
- **Two description types**: Concise alt text vs detailed accessible descriptions
- **SHA256 checksums**: Raw input file checksums for client-side caching
- Adaptive image resizing (text-heavy detection)
- GIF support via FFmpeg conversion to MP4 + Gemini File API
- Batch processing with configurable sizes
- Context-aware descriptions

**Tool:**
`generate_image_descriptions(images, type?, context?, batch_size?, model?)` - Generate descriptions
- `images: list[str]` - Image/GIF paths (1-20)
- `type: str = "alt"` - Description type:
  - `"alt"`: Concise alt text (50-125 chars) - suitable for HTML alt attributes
  - `"description"`: Adaptive descriptions - auto-detects tutorial content for verbose step-by-step narration
- `context: str?` - Document context for more relevant descriptions
- `batch_size: int = 5` - Images per Gemini request (1-10)
- `model: str?` - Gemini model override (default: gemini-flash-latest)

**Description Types:**
- **"alt" (default)**: Concise, focused descriptions for HTML alt attributes (50-125 chars)
- **"description"**: Adaptive descriptions that auto-detect content type:
  - **For UI tutorials/screencasts**: Detailed step-by-step narration with no length limit
    - "Click the 'Save' button", "Select 'Export' from the dropdown"
    - Describes each action: what, where, and result
    - Includes text typed, values entered, transitions
  - **For non-tutorial content**: Detailed but concise (150-300 chars)
    - Spatial layout, colors, textures
    - Visible text, actions, expressions
    - Mood, context, and purpose

**Output Structure:**
```json
{
  "descriptions": {
    "image1.png": "Generated description text"
  },
  "checksums": {
    "image1.png": "sha256_hex_string_64_chars"
  },
  "stats": {
    "total_images": 1,
    "successful": 1,
    "failed": 0,
    "duration_seconds": 2.5
  },
  "metadata": {
    "model": "gemini-flash-latest",
    "description_type": "alt",
    "context_provided": false,
    "checksum_algorithm": "sha256"
  }
}
```

**Dependencies:**
- `google-generativeai`, `google-genai`, `Pillow`
- FFmpeg (optional, for GIF support): `brew install ffmpeg`

## Testing Strategy

Tests are organized by functionality:

**Domain Checker Tests** (`tests/test_domains.py`):
- `TestTLDExtraction` - TLD parsing logic
- `TestChineseCharacterDetection` - Unicode character detection
- `TestNotFoundPatterns` - WHOIS response parsing
- `TestWHOISServerMapping` - Server configuration validation
- `TestOVHVerificationIntegration` - OVH verification response structures, false positive detection, summary formatting
- Integration tests marked with `@pytest.mark.skip` (require mocking)

**Tablica Tests** (`tests/test_tablica.py`):
- `TestPlateValidation` - Polish license plate format validation
- `TestImageOptimization` - Image downscaling and conversion
- `TestHTMLParsing` - Comment extraction from HTML
- `TestHEICSupport` - HEIC format detection
- `TestImageFormats` - PNG, JPEG, grayscale handling
- Integration tests marked with `@pytest.mark.skip` (require HTTP mocking)

**Gemini Image Description Tests** (`tests/test_gemini_alt.py`):
- `TestImageOptimization` - Adaptive image resizing
- `TestContextLoading` - Document context handling
- `TestPromptGeneration` - Prompt creation for alt/description modes
- `TestGenerateImageDescriptionsTool` - Main tool functionality
- `TestGifSupport` - GIF detection (magic bytes, extension)
- `TestGifConversion` - FFmpeg GIF→MP4 conversion
- `TestGifUploadAndGeneration` - Mocked Gemini File API
- `TestGifIntegration` - Full GIF pipeline (requires FFmpeg + GEMINI_API_KEY)
- Integration tests (require GEMINI_API_KEY)

**EXIF Extractor Tests** (`tests/test_exif_extractor.py`):
- `TestConvertToDegrees` - GPS coordinate conversion (DMS to decimal)
- `TestExtractExifData` - EXIF extraction from PNG, JPEG, HEIC
- `TestParseGpsData` - GPS metadata parsing and coordinate extraction
- `TestReverseGeocode` - Nominatim geocoding with mocked responses
- `TestAnalyzeImageMetadataTool` - Main tool functionality (single/batch)
- Integration tests marked with `@pytest.mark.integration` (hit real Nominatim API)

**Plate Recognition Tests** (`tests/test_plate_recognition.py`):
- `TestImageOptimization` - Image downscaling and RGB conversion
- `TestPromptGeneration` - Prompt creation validation
- `TestJSONParsing` - Response parsing (with/without violations)
- `TestMarkdownCodeBlockRemoval` - Cleaning Gemini markdown responses
- `TestRecognizePlatesTool` - Tool functionality (requires mocking)
- Integration tests marked with `@pytest.mark.integration` (require GEMINI_API_KEY)

Test fixtures:
- `tests/fixtures/whois_responses.json` - Sample WHOIS responses
- `tests/fixtures/*.png` - Test images for alt tag generation
- `tests/fixtures/example.gif` - Animated GIF for GIF description testing
- `tests/fixtures/IMG_5134.heic` - iPhone photo with GPS data (Poznań, Poland)
- `tests/fixtures/IMG_2852.heic` - iPhone photo without GPS data

## FastMCP Documentation with Context7

When working with FastMCP, use Context7 MCP tool for up-to-date documentation:
```
# Official FastMCP documentation (most comprehensive)
mcp__Context7__get-library-docs with:
  - context7CompatibleLibraryID="/llmstxt/gofastmcp_llms-full_txt"
  - topic="tools server" (or any specific topic)
  - tokens=2500 (adjust based on needed detail)

# Alternative: GitHub source (if specific implementation needed)
mcp__Context7__get-library-docs with:
  - context7CompatibleLibraryID="/jlowin/fastmcp"
  - topic="your_topic"
```

**Recommended documentation source:** `/llmstxt/gofastmcp_llms-full_txt`
- 12,289 code snippets (official FastMCP documentation)
- Trust Score: 8.0

**Common FastMCP documentation topics:**
- "getting started" - Quickstart and installation
- "decorators context" - Tool/prompt/resource decorators with Context
- "tools server" - Server tool management and patterns
- "testing" - Writing tests for MCP servers
- "deployment" - Deployment configurations
- "client" - Client usage patterns

**Key FastMCP patterns from docs:**
```python
# Tool with Context injection
@mcp.tool
async def process_file(file_uri: str, ctx: Context) -> str:
    ctx.info("Processing file")  # Use context for logging
    return "Result"

# Resource definition
@mcp.resource("resource://{city}/weather")
def get_weather(city: str) -> str:
    return f"Weather for {city}"
```

## Adding New Servers

1. Create `servername.py` with FastMCP instance
2. Implement tools using `@mcp.tool` decorator
3. Add to `[tool.setuptools] py-modules` in pyproject.toml
4. Create corresponding tests in `tests/test_servername.py`
5. Update `.mcp.json` for Claude Code integration

## CI/CD Pipeline

GitHub Actions runs on push to main and PRs:
- Python 3.13 on Ubuntu latest
- Installs project with `pip install -e .`
- Runs pytest with verbose output
- Claude Code integration for PR reviews (claude-code-review.yml)

**GitHub CLI (gh) debugging:**
```bash
# View GitHub Actions run logs (detailed output)
gh run view <RUN_ID> --log

# Extract test failures from logs
gh run view <RUN_ID> --log | grep -A 50 "FAILED\|ERROR\|test session starts" | tail -100

# Watch running workflow
gh run watch <RUN_ID>

# List recent runs
gh run list --limit 10

# Rerun failed jobs
gh run rerun <RUN_ID>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swistaczek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
