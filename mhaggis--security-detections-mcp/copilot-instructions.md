## validation-tools

> Detection validation tools - location, usage, and multi-SIEM support


# Detection Validation Tools

## Per-Platform Validation

### Splunk: contentctl

contentctl validates Splunk detection YAMLs against the security_content schema.

```bash
cd $SECURITY_CONTENT_PATH
source venv/bin/activate
contentctl validate
```

Quick commands:
```bash
# Validate all
contentctl validate

# Validate specific detection
contentctl validate --detection detections/endpoint/your_detection.yml

# Build (generate app from detections)
contentctl build
```

### Sigma: pySigma

pySigma validates Sigma rules and converts to any backend.

```bash
pip install pySigma pySigma-backend-splunk pySigma-backend-microsoft365defender pySigma-backend-elasticsearch

# Validate + convert to Splunk
sigma convert -t splunk -p sysmon rule.yml

# Convert to KQL
sigma convert -t microsoft365defender rule.yml

# Convert to Elastic
sigma convert -t elasticsearch rule.yml
```

### Elastic: detection-rules CLI

Elastic's detection-rules repo has its own validation tooling.

```bash
git clone https://github.com/elastic/detection-rules.git
cd detection-rules
pip install .

# Validate
python -m detection_rules validate-rule path/to/rule.toml
```

### KQL / Sentinel

No standalone CLI validator - use Azure CLI to test analytics rules:

```bash
# Test a KQL query against a workspace
az monitor log-analytics query \
  --workspace <id> \
  --analytics-query "$(cat rule.kql)" \
  --timespan PT1H
```

## Pip Install Requirements

Any pip install MUST use force reinstall or direct PyPI URL (to bypass local registries):

```bash
# Force reinstall
pip install --force-reinstall <package>

# Direct PyPI URL
pip install --index-url https://pypi.org/simple/ <package>

# Both (recommended)
pip install --force-reinstall --index-url https://pypi.org/simple/ <package>
```

## Common Validation Errors (Splunk/contentctl)

### Missing Data Source Fields
```
Data source 'Sysmon EventID 1' has output fields [...] not present in search
```
Fix: Detection SPL must include ALL fields from the data source's `output_fields`.

### File Name Mismatch
```
Expected File Name: xxx.yml
```
Fix: Rename to match snake_case of the `name:` field.

### Missing Story
```
Unable to find: Story Name
```
Fix: Create the story in `stories/` or use an existing story name.

### Invalid RBA Type
Fix: Use only valid RBA `threat_objects` types (process_name, ip_address, file_hash, etc.)

## Common Validation Errors (Sigma)

### Invalid Logsource
Fix: Use standard categories (process_creation, network_connection, file_event, etc.)

### Unsupported Modifier
Fix: Check pySigma docs for supported modifiers per backend.

---
> Source: [MHaggis/Security-Detections-MCP](https://github.com/MHaggis/Security-Detections-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
