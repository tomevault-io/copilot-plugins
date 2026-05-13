## docstrange

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DocStrange is a Python library for extracting and converting documents (PDFs, Word, Excel, PowerPoint, images, URLs) into multiple formats (Markdown, JSON, CSV, HTML) with intelligent content extraction and advanced OCR capabilities.

The library offers two processing modes:
- **Cloud Mode (default)**: Instant conversion using cloud API
- **GPU Mode**: Local processing with GPU acceleration

## Commands

### Development Setup
```bash
# Install in development mode with all dependencies
pip install -e ".[dev]"

# Install with local LLM support (for enhanced JSON extraction)
pip install -e ".[local-llm]"

# Alternative setup script
python scripts/setup_dev.py
```

### Testing
```bash
# Run all tests
python -m pytest tests/ -v

# Run specific test file
python -m pytest tests/test_enhanced_pdf_processor.py -v

# Run with coverage
python -m pytest tests/ --cov=docstrange --cov-report=html
```

### Code Quality
```bash
# Format code with black
black docstrange/ tests/

# Sort imports
isort docstrange/ tests/

# Run linting
flake8 docstrange/ tests/

# Type checking
mypy docstrange/
```

### Building and Distribution
```bash
# Build package
python -m build

# Upload to PyPI (requires credentials)
python -m twine upload dist/*
```

## Architecture

### Core Components

**DocumentExtractor** (`docstrange/extractor.py`)
- Main entry point for document conversion
- Determines processing mode (cloud/cpu/gpu)
- Routes files to appropriate processors
- Handles authentication for cloud mode

**Processor Classes** (`docstrange/processors/`)
- `CloudProcessor`: Handles cloud-based processing via Nanonets API
- `GPUProcessor`: Local GPU-accelerated processing with neural models
- `PDFProcessor`, `DOCXProcessor`, etc.: Format-specific processors
- All processors inherit from `BaseProcessor`

**Pipeline Components** (`docstrange/pipeline/`)
- `NeuralDocumentProcessor`: Core neural processing for local modes
- `LayoutDetector`: Detects document structure and layout
- `OCRService`: Manages OCR engines (EasyOCR, PaddleOCR)
- `NanonetsProcessor`: Cloud API integration

**Services** (`docstrange/services/`)
- `AuthService`: Handles OAuth authentication for cloud mode
- `OllamaService`: Local LLM integration for enhanced JSON extraction

**Result Classes** (`docstrange/result.py`)
- `ConversionResult`: Base result class with extraction methods
- `GPUConversionResult`: Enhanced result for GPU processing
- `CloudConversionResult`: Result wrapper for cloud processing

### Processing Flow

1. **Document Input** → DocumentExtractor.extract()
2. **Mode Selection**: Cloud (default) | CPU | GPU
3. **Format Detection**: Identify file type and route to processor
4. **Processing**:
   - Cloud: Upload to API → Process → Return results
   - Local: Load document → OCR → Layout detection → Structure extraction
5. **Output Generation**: Markdown | JSON | CSV | HTML | Text

### Key Design Patterns

- **Factory Pattern**: DocumentExtractor creates appropriate processor instances
- **Strategy Pattern**: Different processors for different file formats
- **Chain of Responsibility**: OCR fallback mechanism (EasyOCR → PaddleOCR)
- **Caching**: Authentication tokens and model downloads are cached

## Processing Modes

### Cloud Mode (Default)
- No local setup required
- Rate limits: Limited daily calls (free) or 10k/month (authenticated)
- Authentication: `docstrange login` or API key
- Best for: Quick processing without GPU

### GPU Mode  
- Force with `gpu=True` parameter
- Requires CUDA-compatible GPU
- Fastest local processing
- Best for: Batch processing, high-volume workloads

## Authentication & Rate Limits

### Free Tier
- Limited daily API calls
- No authentication required

### Authenticated Access (10k docs/month)
```bash
# Browser-based login (recommended)
docstrange login

# Check status
docstrange --login

# Logout
docstrange --logout
```

### API Key Access (10k docs/month)
- Get key from https://app.nanonets.com/#/keys
- Pass via `api_key` parameter or `NANONETS_API_KEY` env var

## MCP Server Integration

The repository includes an MCP server for Claude Desktop integration (local development only):

### Setup
1. Install: `pip install -e ".[dev]"`
2. Configure in `~/Library/Application Support/Claude/claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "docstrange": {
      "command": "python3",
      "args": ["/path/to/docstrange/mcp_server_module/server.py"]
    }
  }
}
```

### Key Features
- Token-aware document processing
- Hierarchical navigation for large documents
- Smart chunking based on token limits
- Document search and section extraction

## Dependencies

### Core Dependencies
- `pdf2image`: PDF to image conversion
- `python-docx`, `python-pptx`, `openpyxl`: Office formats
- `beautifulsoup4`, `markdownify`: HTML/Markdown conversion
- `Pillow`, `pdf2image`: Image processing

### ML/OCR Dependencies
- `easyocr`: Primary OCR engine
- `paddleocr`: Fallback OCR (optional)
- `docling-ibm-models`: Layout detection
- `transformers`, `huggingface_hub`: Model management

### Optional Dependencies
- `ollama`: Local LLM for enhanced JSON extraction
- `mcp`, `tiktoken`: MCP server support (Python 3.10+)

## Environment Variables

- `NANONETS_API_KEY`: API key for cloud processing
- `OLLAMA_HOST`: Ollama server URL (default: http://localhost:11434)
- `HF_HOME`: Hugging Face cache directory for models

## Common Tasks

### Extract specific fields from documents
```python
result = extractor.extract("invoice.pdf")
fields = result.extract_data(specified_fields=["invoice_number", "total_amount"])
```

### Process with JSON schema
```python
schema = {"invoice_number": "string", "total_amount": "number"}
structured = result.extract_data(json_schema=schema)
```

### Force local processing
```python
# GPU mode (requires CUDA)
extractor = DocumentExtractor(gpu=True)
```

## Error Handling

The library uses custom exceptions:
- `ConversionError`: General conversion failures
- `UnsupportedFormatError`: Unknown file format
- `FileNotFoundError`: Missing input file

Cloud mode automatically retries on transient failures.
Local modes fall back through OCR engines if one fails.

## How to code
- If you are making any frontend changes, always try to use playwright to test changes to see if what you have implemented actually works in web

---
> Source: [NanoNets/docstrange](https://github.com/NanoNets/docstrange) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
