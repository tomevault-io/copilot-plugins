## contentctl-path

> contentctl tool location and usage (Splunk-specific). For other SIEM validation tools, see validation-tools.mdc.


# contentctl Tool Configuration (Splunk-Specific)

> **Note**: contentctl is Splunk's validation tool for security_content YAMLs. For non-Splunk platforms, see the **validation-tools.mdc** rule which covers Sigma (pySigma), Elastic (detection-rules CLI), and Sentinel (Azure CLI).

## Location

contentctl is installed in the LOCAL VENV within the security_content repo:

```bash
cd $SECURITY_CONTENT_PATH
source venv/bin/activate
contentctl validate
```

## Pip Install Requirements

Any pip install MUST use force reinstall or direct PyPI URL:

```bash
# Option 1: Force reinstall
pip install --force-reinstall <package>

# Option 2: Direct PyPI URL
pip install --index-url https://pypi.org/simple/ <package>

# Option 3: Both (recommended)
pip install --force-reinstall --index-url https://pypi.org/simple/ <package>
```

## Quick Commands

```bash
# Validate all content
cd $SECURITY_CONTENT_PATH && source venv/bin/activate && contentctl validate

# Validate specific detection
cd $SECURITY_CONTENT_PATH && source venv/bin/activate && contentctl validate --detection detections/endpoint/your_detection.yml

# Create new detection (interactive)
cd $SECURITY_CONTENT_PATH && source venv/bin/activate && contentctl new --type detection

# Build
cd $SECURITY_CONTENT_PATH && source venv/bin/activate && contentctl build
```

## NOT These Paths (Wrong!)

- ❌ `<wrong-venv>/bin/ba_contentctl` - Different tool
- ❌ Global `pip install contentctl` - Not installed globally
- ❌ Attack Range venv - Doesn't have contentctl

## Common Validation Errors

### Missing Data Source Fields
```
Data source 'Sysmon for Linux EventID 1' has output fields [...] not present in search
```
**Fix**: The detection's SPL must include ALL fields listed by the data source. Check `data_sources/` for required fields.

### File Name Mismatch
```
Expected File Name: xxx.yml
Actual File Name: yyy.yml
```
**Fix**: Rename file to match the `name:` field (lowercase, underscores).

### Missing Story
```
Unable to find: Story Name
```
**Fix**: Either create the story in `stories/` or use an existing story name.

### Invalid RBA Type
```
Invalid Value for rba - Valid options: 'system', 'user', 'ip_address', ...
```
**Fix**: Use only valid RBA threat_objects types from the allowed list.

---
> Source: [MHaggis/Security-Detections-MCP](https://github.com/MHaggis/Security-Detections-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
