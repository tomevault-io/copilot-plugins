## netdata-was110-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Netdata external plugin for monitoring BFW Solutions WAS-110 XGS-PON SFP+ modules running the 8311 community firmware. The plugin collects optical power, temperature, PLOAM state, and electrical metrics via the device's JSON API endpoint.

## Key Files

- **`was110.plugin`**: Main Python 3 plugin executable
- **`was110.conf`**: Configuration file template with device settings and thresholds
- **`was110-test.sh`**: Shell script for testing API connectivity and plugin functionality
- **`was110-alarms.conf`**: Netdata health alarm definitions
- **`README.md`**: Installation and configuration guide for FreeBSD

## Development Commands

### Testing the Plugin
```bash
# Test plugin standalone (2-second updates)
./was110.plugin 2

# Test API connectivity
./was110-test.sh

# Test as netdata user
sudo -u netdata ./was110.plugin 2
```

### Installation Commands
```bash
# Install plugin
sudo cp was110.plugin /usr/libexec/netdata/plugins.d/
sudo chmod +x /usr/libexec/netdata/plugins.d/was110.plugin

# Install configuration
sudo cp was110.conf /etc/netdata/
sudo chown netdata:netdata /etc/netdata/was110.conf

# Install alarms
sudo cp was110-alarms.conf /etc/netdata/health.d/

# Restart Netdata
sudo service netdata restart
```

### Debugging Commands
```bash
# Check Netdata logs
sudo tail -f /var/log/netdata/error.log | grep was110

# Verify plugin is running
ps aux | grep was110.plugin

# Test manual API call
curl -k -u root:8311 https://192.168.11.1/cgi-bin/luci/8311/metrics
```

## Architecture

### Plugin Design
- **Language**: Python 3 (FreeBSD compatible)
- **Protocol**: Netdata external plugin format (stdout-based)
- **Update Interval**: 2 seconds (configurable)
- **Authentication**: HTTP Basic Auth with self-signed SSL cert

### Chart Structure
The plugin outputs 7 charts in the `was110` family:

1. **`ploam_state`**: XGS-PON connection state (51=O5.1, 52=O5.2, etc.)
2. **`optical_power`**: RX/TX power in dBm (stored as hundredths)
3. **`signal_quality`**: Percentage based on RX power thresholds
4. **`temperature`**: Optic, CPU1, CPU2 temperatures (stored as hundredths)
5. **`electrical`**: Module voltage and TX bias current
6. **`connection_uptime`**: Seconds in operational state
7. **`state_transitions`**: Event counters for PLOAM transitions

### Data Processing
- Values with decimals stored as integers (multiplied by 100) for precision
- PLOAM state changes trigger logging and counter updates
- Signal quality calculated from RX power vs configurable thresholds
- Connection uptime resets when leaving O5 operational states

### Error Handling
- Network timeouts: Output zeros, continue monitoring
- Authentication errors: Log and retry
- JSON parse errors: Skip update cycle
- SSL certificate errors: Expected (self-signed), verification disabled

## Configuration

### Device Settings (`was110.conf`)
- **URL**: WAS-110 API endpoint (usually `https://192.168.11.1/cgi-bin/luci/8311/metrics`)
- **Credentials**: HTTP Basic Auth (default: root/8311)
- **Timeouts**: HTTP request timeout (default: 5 seconds)
- **Thresholds**: RX power quality levels for AT&T fiber

### Signal Quality Thresholds
Based on real-world AT&T XGS-PON experience:
- **Excellent** (100%): ≥ -21 dBm
- **Good** (80%): ≥ -25 dBm
- **Acceptable** (60%): ≥ -27 dBm
- **Poor** (40%): ≥ -30 dBm
- **Critical** (20%): < -30 dBm

## PLOAM State Reference

| State | Name | Description |
|-------|------|-------------|
| 51 | O5.1 Associated | Normal operation |
| 52 | O5.2 Operational | Also good |
| 10 | O1 Initial | Just powered on |
| 20/30 | O2-O3 Serial Number | Authenticating |
| 40 | O4 Ranging | Distance measurement |
| 60 | O6 POPUP | Signal loss, recovering |
| 70 | O7 Emergency Stop | Laser disabled |

## Platform Specifics

### FreeBSD Requirements
- Python 3.9+ with `requests` and `urllib3` modules
- Netdata installed via ports or packages
- Plugin runs as `netdata` user
- Configuration files owned by `netdata:netdata`

### Dependencies
```bash
pkg install python39 py39-pip
pip install requests urllib3
```

## Monitoring and Alarms

### Critical Alarms
- PLOAM state != 51/52 (connection down)
- RX power < -30 dBm (signal critical)
- Frequent state transitions (unstable connection)
- Plugin not updating (stale data)

### Warning Conditions
- RX power -27 to -30 dBm (degrading signal)
- High temperatures (>70°C optic, >80°C CPU)
- Abnormal voltage or TX bias levels

## Testing Strategy

1. **API Connectivity**: `was110-test.sh` validates network, auth, JSON response
2. **Plugin Output**: Verify Netdata format compliance
3. **Chart Generation**: Confirm all 7 charts appear in dashboard
4. **State Transitions**: Test PLOAM state change detection
5. **Error Handling**: Validate graceful degradation on API failures

## Troubleshooting

### Common Issues
- **SSL Certificate Warnings**: Expected (self-signed), verification disabled
- **Permission Errors**: Check file ownership and execute permissions
- **Authentication Failures**: Verify credentials in config file
- **Missing Charts**: Check plugin output format and Netdata restart
- **High Memory Usage**: Plugin is stateless, minimal resource usage expected

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/csmarshall)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/csmarshall)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
