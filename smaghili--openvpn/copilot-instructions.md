## openvpn

> Traffic monitoring system guidelines and patterns


## 👨‍💻 Principal Developer Standards
- **System Designer Mindset**: Always consider impact on overall architecture
- **Performance First**: Optimize for low-resource servers
- **Clean Code**: No inline comments, English docstrings only
- **Risk Assessment**: Alert about potential monitoring system risks

# Traffic Monitoring System Rules

## 🔍 Monitoring Architecture

The traffic monitoring system is the **core feature** of this OpenVPN manager. It provides real-time quota enforcement and traffic logging.

### Key Components
- **Monitor Service**: [scripts/monitor_service.py](mdc:scripts/monitor_service.py) - Main monitoring daemon
- **Connection Hook**: [scripts/on_connect.py](mdc:scripts/on_connect.py) - Pre-connection quota check
- **Disconnection Hook**: [scripts/on_disconnect.py](mdc:scripts/on_disconnect.py) - Traffic recording

## ⚡ Critical Monitoring Rules

### Real-time Processing
- Monitor service connects to **both** certificate and login server management interfaces
- Use separate socket connections for each server type
- Parse `CLIENT_LIST` entries from OpenVPN status output
- Combine traffic data when users connect to multiple servers

### Quota Enforcement Strategy
1. **Pre-connection check**: Block users who already exceeded quota
2. **Real-time monitoring**: Track incremental traffic and disconnect violators
3. **Post-disconnection recording**: Update database with session traffic

### Error Resilience
- Implement exponential backoff for connection failures
- Handle socket timeouts gracefully
- Continue monitoring even if one server connection fails
- Log all errors with timestamps and context

### Performance Optimization
- Track incremental traffic changes (not absolute values)
- Clean up disconnected user tracking data
- Use efficient CLIENT_LIST parsing
- Implement log rotation to prevent disk space issues

## 🛡️ Data Integrity
- Use database transactions for traffic updates
- Validate traffic data before recording
- Handle concurrent access to user quota data
- Maintain audit trail in `traffic_logs` table

### Configuration
- All monitoring settings in [environment.env](mdc:environment.env)
- Support for configurable check intervals
- Flexible log file locations
- Database path configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smaghili) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
