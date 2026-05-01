## detection-validation

> > **Note**: This file primarily covers Splunk (contentctl) validation rules. For validation across all SIEM platforms, see the `validation-tools.mdc` rule and the `/siem-validator` agent.

# Detection Validation - Critical Rules

> **Note**: This file primarily covers Splunk (contentctl) validation rules. For validation across all SIEM platforms, see the `validation-tools.mdc` rule and the `/siem-validator` agent.

## Test Section Format (CRITICAL - Splunk)

The `tests:` section MUST have valid URLs, NOT placeholder descriptions:

```yaml
# CORRECT
tests:
  - name: True Positive Test
    attack_data:
      - data: 
          https://media.githubusercontent.com/media/<your-org>/attack_data/master/datasets/malware/chaos_ransomware/sysmon.log
        source: XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
        sourcetype: XmlWinEventLog

# WRONG - will fail validation
tests:
  - name: True Positive Test
    attack_data:
      - data: Custom test data required - some description here
        source: XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
        sourcetype: XmlWinEventLog
        description: This field is NOT allowed
```

**Rules:**
- `data:` MUST be a URL or valid file path
- `description:` field is NOT allowed in attack_data
- Use existing attack_data from similar detections when custom data doesn't exist

## RBA Types (CRITICAL)

Valid `threat_objects` types:
- certificate_common_name, certificate_organization, certificate_serial, certificate_unit
- command, domain, email_address, email_subject
- file_hash, file_name, file_path, http_user_agent
- ip_address, process, process_name, parent_process, parent_process_name, process_hash
- registry_path, registry_value_name, registry_value_text
- service, signature, system, tls_hash, url

Valid `risk_objects` types:
- system, user, ip_address (fewer options than threat_objects)

**Common Mistakes:**
- `file_path` is NOT a valid risk_objects type (use `system` instead)
- Don't use `score` in threat_objects

## Data Source Output Fields (CRITICAL)

When using a data_source, your search MUST output ALL fields listed in the data source's `output_fields:`.

**Example:** Sysmon EventID 3 requires these fields in output:
- action, app, dest, dest_ip, dest_port, direction, dvc, protocol
- protocol_version, src, src_ip, src_port, transport, user, vendor_product

**Fix:** Either:
1. Add all required fields to your stats/by clause
2. Use eval to create missing fields with default values
3. Use a CIM data model (Network_Traffic) instead of raw Sysmon

## Filter Macro Naming

Filter macro MUST be: `snake_case(detection_name)_filter`

```yaml
name: Windows Cobalt Strike Beacon Watermark 666
# Filter macro must be:
| `windows_cobalt_strike_beacon_watermark_666_filter`

# NOT:
| `windows_cobalt_strike_beacon_license_id_666_filter`  # WRONG
```

## Story References

`analytic_story` must reference stories that EXIST in `stories/` folder.

Before adding: `analytic_story: [My New Story]`
First create: `stories/my_new_story.yml`

## File Naming

File name MUST match snake_case of detection name:
- name: `Windows Embargo Ransomware File Extensions`
- file: `windows_embargo_ransomware_file_extensions.yml`

## Quick Validation

Run the appropriate validator before committing:

**Splunk**:
```bash
cd $SECURITY_CONTENT_PATH
source venv/bin/activate && contentctl validate
```

**Sigma**:
```bash
sigma convert -t splunk -p sysmon rule.yml  # Test conversion
```

**Elastic**:
```bash
cd detection-rules && python -m detection_rules validate-rule path/to/rule.toml
```

**KQL / Sentinel**:
```bash
az monitor log-analytics query --workspace <id> --analytics-query "$(cat rule.kql)" --timespan PT1H
```

## Finding Valid Attack Data URLs

Search existing detections for attack_data that matches your data source:
```bash
rg "attack_data.*T1547" detections/
rg "datasets.*sysmon" detections/endpoint/
```

Common reusable test data:
- Chaos ransomware (startup folder): `datasets/malware/chaos_ransomware/sysmon.log`
- Cobalt Strike: `datasets/attack_techniques/T1055/cobalt_strike/windows-sysmon.log`
- PowerShell: `datasets/attack_techniques/T1059.001/*/`

---
> Source: [MHaggis/Security-Detections-MCP](https://github.com/MHaggis/Security-Detections-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
