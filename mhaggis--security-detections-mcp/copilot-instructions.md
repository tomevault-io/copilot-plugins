## linux-testing-workflow

> Linux detection testing workflow and lessons learned


# Linux Detection Testing Workflow

## Key Learnings

### 1. Parent Process Relationships Are Hard to Simulate

When testing detections that require specific parent-child process relationships:

**Challenge**: Sysmon captures the actual binary path, not script names
- Running `/tmp/telnetd` (a bash script) shows ParentImage: `/usr/bin/bash`
- `exec -a telnetd bash` shows CommandLine with `telnetd` but Image still `/usr/bin/bash`

**Solutions**:
1. **Use the actual service**: If possible, configure the real daemon (e.g., xinetd → telnetd)
2. **Compile a binary**: Create a C program named exactly as needed
3. **Accept partial match**: Generate command-line telemetry even if parent doesn't match exactly

### 2. Patched vs Vulnerable Behavior

Modern packages are patched - they won't reproduce vulnerable behavior:
- Installed `inetutils-telnetd` is patched for CVE-2026-24061
- It won't pass `-f root` to login even with USER environment variable

**Solutions**:
1. Create synthetic test data that matches expected pattern
2. Use attack_data repo pre-built datasets
3. Document that live testing requires vulnerable binary

### 3. Sysmon for Linux Data Flow

The data flow depends on your SIEM platform:

**Splunk**:
```
Linux Host → Sysmon → journald → Splunk Universal Forwarder → Splunk
```
- Data lands in `unix` index by default, sourcetype: `sysmon:linux`

**Sentinel**:
```
Linux Host → Sysmon → journald → Azure Monitor Agent (AMA) → Log Analytics Workspace
```
- Data lands in `Syslog` table or custom table via DCR

**Elastic**:
```
Linux Host → Sysmon → journald → Elastic Agent / Filebeat → Elasticsearch
```
- Data lands in `logs-system.syslog-*` or custom index

**Common**:
- Process creation: EventCode=1 / EventID=1
- Ingestion delay: 1-3 minutes typical (all platforms)

### 4. Testing Checklist

Before claiming validation:
- [ ] Data exists in target SIEM (check index/table/data stream)
- [ ] Detection query returns results
- [ ] Field values match expected pattern
- [ ] Parent-child relationship correct (if required)
- [ ] Severity/risk score appropriate

## Linux Attack Range Access

```bash
# SSH to Linux target
ssh -i ~/.ssh/id_ar ubuntu@<IP>

# Check Sysmon running
ps aux | grep sysmon

# View recent Sysmon events
sudo journalctl -u sysmon --since "5 minutes ago" | head -50

# Check specific EventCode
sudo journalctl -u sysmon --since "5 minutes ago" | grep "EventID>1<"
```

## Data Location by SIEM

### Splunk Sourcetype Mappings
| Source | Sourcetype | Index |
|--------|------------|-------|
| Sysmon Linux | sysmon:linux | unix |
| Linux Audit | linux:audit | unix |
| Windows Sysmon | XmlWinEventLog:Microsoft-Windows-Sysmon/Operational | win |

### Sentinel Table Mappings
| Source | Table |
|--------|-------|
| Sysmon Linux | Syslog (or SysmonEvent via custom DCR) |
| Linux Audit | CommonSecurityLog or Syslog |
| Windows (MDE) | DeviceProcessEvents |

### Elastic Index Mappings
| Source | Index Pattern |
|--------|---------------|
| Sysmon Linux | logs-system.syslog-* |
| Linux Audit | logs-system.auth-* |
| Windows (Endpoint Security) | logs-endpoint.events.process-* |

## Story Alignment

When creating complementary detections for a PR:
1. Check the existing story name in the PR
2. Either use the SAME story or create a new one and reference both
3. Don't create duplicate stories with different names

Example issue found:
- PR story: "Telnetd CVE-2026-24061"  
- Our story: "CVE-2026-24061 Telnetd Authentication Bypass"

Fix: Align story names or update detections to reference both.

---
> Source: [MHaggis/Security-Detections-MCP](https://github.com/MHaggis/Security-Detections-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
