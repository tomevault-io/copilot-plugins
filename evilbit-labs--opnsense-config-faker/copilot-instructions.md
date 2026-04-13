## opnsense-config-faker

> **Note**: This document applies only to the legacy Python implementation preserved in the `legacy/` directory and the Python-based XSD model generation tools. The primary project implementation is in Rust.


# Python Development Standards for Legacy Code and XSD Tooling

**Note**: This document applies only to the legacy Python implementation preserved in the `legacy/` directory and the Python-based XSD model generation tools. The primary project implementation is in Rust.

## Scope of Python Usage

Python is used in this project for:

1. **Legacy Implementation Reference**: Original Python code in `legacy/` directory (read-only)
2. **XSD Model Generation**: Using xsdata-pydantic to generate type-safe models from XSD schema
3. **Schema Validation Tooling**: Python scripts for validating generated XML against XSD
4. **Development Utilities**: Build and validation scripts that complement the Rust toolchain

## Project Setup and Dependencies

### Package Management

- **Primary**: Use **UV** package manager for modern dependency management
- **Python Version**: Python 3.10+ required, 3.13+ recommended for best performance
- **Virtual Environment**: Always use isolated environments for Python dependencies

```bash
# Install UV if not available
curl -LsSf https://astral.sh/uv/install.sh | sh

# Setup project with Python dependencies
uv sync --extra dev

# Install xsdata-pydantic for XSD model generation
uv add xsdata-pydantic

# Install additional development tools
uv add --dev mypy black isort pytest
```

### Core Dependencies

**XSD and Validation Tools:**

- **xsdata-pydantic**: For XSD-based model generation with Pydantic validation
- **lxml**: For XML processing and validation
- **pydantic**: For data validation and serialization (used by generated models)
- **xmlschema**: Alternative XSD validation library

**Development Tools:**

- **mypy**: Static type checking
- **black**: Code formatting (119 character line length to match project)
- **isort**: Import organization
- **pytest**: Testing framework

## Code Style and Quality

### Formatting Standards

```toml
# pyproject.toml - Python configuration
[tool.black]
line-length = 119  # Match Rust project standards
target-version = ['py310']
include = '\.pyi?$'
extend-exclude = '''
/(
  # Exclude generated model files
  opnsense/models/.*\.py
)/
'''

[tool.isort]
profile = "black"
line_length = 119
skip_glob = ["opnsense/models/*"]

[tool.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
exclude = [
    "^opnsense/models/.*\\.py$",  # Skip generated models
    "^legacy/.*\\.py$",           # Skip legacy code
]
```

### Type Safety Requirements

All Python code must use comprehensive type hints:

```python
from typing import Optional, List, Dict, Any, Union
from pathlib import Path
import xml.etree.ElementTree as ET

def validate_opnsense_config(
    xml_file: Path,
    xsd_schema: Path,
    strict: bool = True
) -> tuple[bool, List[str]]:
    """Validate OPNsense XML configuration against XSD schema.

    Args:
        xml_file: Path to the XML configuration file
        xsd_schema: Path to the XSD schema file
        strict: Whether to enforce strict validation rules

    Returns:
        Tuple of (is_valid, list_of_errors)

    Raises:
        FileNotFoundError: If xml_file or xsd_schema doesn't exist
        XMLSyntaxError: If XML is malformed
    """
    errors: List[str] = []

    try:
        # Implementation with proper error handling
        from lxml import etree

        with open(xsd_schema, 'r') as schema_file:
            schema_doc = etree.parse(schema_file)
            schema = etree.XMLSchema(schema_doc)

        with open(xml_file, 'r') as xml_doc:
            xml_tree = etree.parse(xml_doc)

        is_valid = schema.validate(xml_tree)
        if not is_valid:
            errors = [str(error) for error in schema.error_log]

        return is_valid, errors

    except FileNotFoundError as e:
        raise FileNotFoundError(f"Required file not found: {e}")
    except etree.XMLSyntaxError as e:
        raise etree.XMLSyntaxError(f"Malformed XML: {e}")
```

## XSD Model Generation Integration

### xsdata-pydantic Usage

```python
# scripts/generate_models.py - Model generation script
from pathlib import Path
import subprocess
import sys
from typing import List, Optional

def generate_pydantic_models(
    xsd_file: Path,
    output_package: str = "opnsense.models",
    config_file: Optional[Path] = None
) -> bool:
    """Generate Pydantic models from XSD schema using xsdata.

    Args:
        xsd_file: Path to the XSD schema file
        output_package: Python package name for generated models
        config_file: Optional xsdata configuration file

    Returns:
        True if generation successful, False otherwise
    """
    if not xsd_file.exists():
        print(f"❌ XSD file not found: {xsd_file}")
        return False

    cmd: List[str] = [
        "xsdata",
        str(xsd_file),
        "--output", "pydantic",
        "--package", output_package,
        "--structure-style", "single-package"
    ]

    if config_file and config_file.exists():
        cmd.extend(["--config", str(config_file)])

    try:
        result = subprocess.run(cmd, check=True, capture_output=True, text=True)
        print(f"✅ Models generated successfully in {output_package}")
        return True

    except subprocess.CalledProcessError as e:
        print(f"❌ Model generation failed: {e.stderr}")
        return False

if __name__ == "__main__":
    xsd_path = Path("opnsense-config.xsd")
    config_path = Path("xsdata-config.xml")

    success = generate_pydantic_models(
        xsd_path,
        config_file=config_path if config_path.exists() else None
    )

    sys.exit(0 if success else 1)
```

### Generated Model Usage

```python
# Example of using generated models for validation
from opnsense.models import Opnsense, Vlans, Vlan
from typing import List
from pathlib import Path

def create_sample_config() -> Opnsense:
    """Create a sample OPNsense configuration using generated models."""
    vlans = [
        Vlan(
            if_="em0",
            tag="100",
            descr="IT_VLAN_0100",
            vlanif="em0_vlan100"
        ),
        Vlan(
            if_="em0",
            tag="101",
            descr="Sales_VLAN_0101",
            vlanif="em0_vlan101"
        )
    ]

    return Opnsense(
        vlans=Vlans(vlan=vlans),
        # ... other configuration sections
    )

def validate_generated_xml(xml_content: str) -> bool:
    """Validate XML content using generated Pydantic models."""
    try:
        # Parse XML and validate using Pydantic models
        from lxml import etree
        tree = etree.fromstring(xml_content.encode())

        # Convert to Pydantic model for validation
        config = Opnsense.from_xml(xml_content)  # If xsdata supports XML parsing

        # Pydantic validation happens automatically during creation
        return True

    except Exception as e:
        print(f"Validation failed: {e}")
        return False
```

## Testing Standards for Python Code

### Test Structure

```python
# tests/test_xsd_validation.py
import pytest
from pathlib import Path
import tempfile
from lxml import etree

from scripts.generate_models import generate_pydantic_models
from scripts.validate_xml import validate_opnsense_config

@pytest.fixture
def sample_xsd() -> Path:
    """Provide path to test XSD schema."""
    return Path("opnsense-config.xsd")

@pytest.fixture
def sample_xml() -> str:
    """Provide sample OPNsense XML configuration."""
    return '''<?xml version="1.0" encoding="UTF-8"?>
    <opnsense>
        <vlans>
            <vlan>
                <if>em0</if>
                <tag>100</tag>
                <descr>Test VLAN</descr>
                <vlanif>em0_vlan100</vlanif>
            </vlan>
        </vlans>
    </opnsense>'''

def test_xsd_validation_with_valid_xml(sample_xsd: Path, sample_xml: str):
    """Test XSD validation with valid XML."""
    with tempfile.NamedTemporaryFile(mode='w', suffix='.xml', delete=False) as f:
        f.write(sample_xml)
        xml_path = Path(f.name)

    try:
        is_valid, errors = validate_opnsense_config(xml_path, sample_xsd)
        assert is_valid, f"Validation failed with errors: {errors}"
        assert not errors, "No errors should be present for valid XML"
    finally:
        xml_path.unlink()

def test_model_generation(sample_xsd: Path):
    """Test that XSD model generation completes successfully."""
    success = generate_pydantic_models(sample_xsd)
    assert success, "Model generation should succeed"

    # Verify generated files exist
    models_dir = Path("opnsense/models")
    assert models_dir.exists(), "Models directory should be created"

    init_file = models_dir / "__init__.py"
    assert init_file.exists(), "Package init file should be created"

@pytest.mark.skipif(not Path("opnsense/models").exists(),
                   reason="Generated models not available")
def test_generated_model_import():
    """Test that generated models can be imported and used."""
    try:
        from opnsense.models import Opnsense, Vlans, Vlan

        # Create a simple configuration using generated models
        vlan = Vlan(if_="em0", tag="100", descr="Test")
        vlans = Vlans(vlan=[vlan])
        config = Opnsense(vlans=vlans)

        # Basic validation that objects were created
        assert config.vlans is not None
        assert len(config.vlans.vlan) == 1
        assert config.vlans.vlan[0].tag == "100"

    except ImportError as e:
        pytest.fail(f"Failed to import generated models: {e}")
```

## Integration with Rust Toolchain

### Justfile Integration

Python scripts should integrate cleanly with the Rust-based Just commands:

```makefile
# Justfile tasks for Python tooling

# Verify Python XSD tooling is available
verify-python-xsd:
    @command -v uv >/dev/null 2>&1 || (echo "❌ uv not found. Install from https://astral.sh/uv/" && exit 1)
    @uv run python -c "import xsdata" || (echo "❌ xsdata not found. Run: uv add xsdata-pydantic" && exit 1)
    @echo "✅ Python XSD tooling verified"

# Generate XSD models using Python
generate-models: verify-python-xsd
    uv run python scripts/generate_models.py
    @echo "✅ XSD models generated successfully"

# Validate XML against XSD using Python
validate-xml FILE: verify-python-xsd
    uv run python scripts/validate_xml.py {{FILE}}
    @echo "✅ XML validation completed"

# Run Python tests
test-python: verify-python-xsd
    uv run pytest tests/ -v
    @echo "✅ Python tests completed"

# Format Python code
format-python:
    uv run black .
    uv run isort .
    @echo "✅ Python code formatted"

# Type check Python code
typecheck-python:
    uv run mypy .
    @echo "✅ Python type checking completed"
```

## Error Handling and Logging

### Error Handling Patterns

```python
import logging
from typing import Union, NoReturn
from pathlib import Path

# Configure logging for Python utilities
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class XSDValidationError(Exception):
    """Custom exception for XSD validation failures."""

    def __init__(self, message: str, errors: List[str] = None):
        super().__init__(message)
        self.errors = errors or []

def handle_validation_error(e: Exception, file_path: Path) -> NoReturn:
    """Centralized error handling for validation operations."""
    logger.error(f"Validation failed for {file_path}: {e}")

    if isinstance(e, XSDValidationError) and e.errors:
        logger.error("Validation errors:")
        for error in e.errors:
            logger.error(f"  • {error}")

    sys.exit(1)
```

## Legacy Code Preservation

### Guidelines for Legacy Directory

- **Read-Only**: Never modify code in `legacy/` directory
- **Attribution**: Preserve original author attribution and licensing
- **Documentation**: Document the relationship between legacy and current implementations
- **Reference**: Use legacy code as reference for understanding original functionality

```python
# legacy/README.md should contain:
"""
# Legacy Python Implementation

This directory contains the original Python implementation of OPNsense-config-faker
for reference purposes only. This code should NOT be modified or used in the
current application.

## Original Attribution

This code was originally developed by [original authors] and is preserved here
to maintain historical context and assist with the transition to the Rust-based
implementation.

## Usage

DO NOT USE THIS CODE DIRECTLY. It is maintained purely for:
- Understanding original functionality
- Comparing implementation approaches
- Preserving project history

For current functionality, use the Rust implementation in the `src/` directory.
"""
```

This Python integration approach ensures that XSD tooling and legacy code are properly managed while maintaining the focus on the Rust-based primary implementation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EvilBit-Labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
