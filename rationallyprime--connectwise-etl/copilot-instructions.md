## connectwise-etl

> ConnectWise ETL Framework for Microsoft Fabric OneLake with medallion architecture.

# CLAUDE.md

ConnectWise ETL Framework for Microsoft Fabric OneLake with medallion architecture.

## Architecture

### Bronze Layer
- Pydantic validation during API extraction
- Store validated raw data with original structure
- API data small enough for row-by-row validation

### Silver Layer
- NO validation (already done in Bronze)
- Apply SparkDantic schema from models
- Flatten nested columns preserving CamelCase
- Pure schema-driven type conversion

### Gold Layer
- Add surrogate keys via window functions
- Business-specific fact tables
- Icelandic agreement type handling

## Model Generation

### CRITICAL: Use these exact flags
```bash
uv run datamodel-codegen \
    --input PSA_OpenAPI_schema_patched.json \
    --output src/connectwise_etl/models/ \
    --input-file-type openapi \
    --base-class sparkdantic.SparkModel \
    --output-model-type pydantic_v2.BaseModel \
    --parent-scoped-naming \
    --reuse-model \
    --keep-model-order \
    --use-annotated \
    --use-standard-collections \
    --force-optional \
    --use-union-operator \
    --enum-field-as-literal all \
    --openapi-scopes schemas \
    --disable-timestamp
```

### DO NOT USE
- `--use-subclass-enum` (breaks SparkDantic)
- `--snake-case-field` (must preserve CamelCase)
- Single file output (must use directory)

### Post-Generation Requirements
Models MUST be rebuilt to resolve forward references:
```python
# In models/registry.py
Agreement.model_rebuild()
TimeEntry.model_rebuild()
Invoice.model_rebuild()
ExpenseTax.model_rebuild()  # ExpenseEntry depends on this
ExpenseEntry.model_rebuild()
ProductItem.model_rebuild()
Member.model_rebuild()
Company.model_rebuild()
```

## Schema Patches

`PSA_OpenAPI_schema_patched.json` fixes:
- customFields.value accepts string|object (not just object)
- Removed non-existent postedinvoice endpoint

## Key Patterns

### Fail-Fast Philosophy
ALL parameters required, no defaults:
```python
# WRONG: def create_fact(df, config=None)
# RIGHT: def create_fact(df: DataFrame, config: dict[str, Any])
```

### Model Naming
No underscores in entity names:
```python
models = {
    "timeentry": TimeEntry,    # NOT "time_entry"
    "expenseentry": ExpenseEntry  # NOT "expense_entry"
}
```

### Icelandic Agreement Types
- **yÞjónusta**: Billable service
- **Tímapottur**: Prepaid hours (excluded from invoices)
- **Innri verkefni**: Internal projects
- **Rekstrarþjónusta/Alrekstur**: Operations/maintenance

Pattern: `r"Tímapottur\s*:?"`

## Environment Variables

Required for ConnectWise API:
- `CW_AUTH_USERNAME`
- `CW_AUTH_PASSWORD`
- `CW_CLIENTID`

`.env` format for Python scripts:
```python
os.environ["CW_AUTH_USERNAME"] = "value"
os.environ["CW_AUTH_PASSWORD"] = "value"
os.environ["CW_CLIENTID"] = "value"
```

## Testing

### Local validation test
```bash
uv run python test_client_extraction.py
```

### Build wheel for Fabric
```bash
uv build --wheel
```

### Fabric deployment
1. Upload `dist/connectwise_etl-*.whl` to lakehouse
2. Install: `%pip install /lakehouse/default/Files/dist/connectwise_etl-*.whl`
3. Run: `from connectwise_etl.main import run_etl_pipeline; run_etl_pipeline()`

## Project Structure

```
src/connectwise_etl/
├── client.py         # API client with field selection
├── main.py          # Pipeline orchestration (config eliminated)
├── transforms.py    # Fact creators
├── models/
│   ├── __init__.py  # Auto-generated, DO NOT EDIT
│   └── registry.py  # Model registry with rebuilds
└── utils/           # Logging, exceptions, decorators
```

## Common Issues

1. **TypeConversionError**: Models not rebuilt - add to registry.py
2. **Validation failures**: Check PSA_OpenAPI_schema_patched.json
3. **Forward references**: Call model_rebuild() after import
4. **Enum errors**: Must use `--enum-field-as-literal all` flag

---
> Source: [RationallyPrime/connectwise-etl](https://github.com/RationallyPrime/connectwise-etl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
