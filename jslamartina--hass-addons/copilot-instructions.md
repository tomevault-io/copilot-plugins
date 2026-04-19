## hass-addons

> Logging patterns and examples


# Logging Examples

## Basic Patterns

```python
## Function entry with context (structured logging)
logger.info(
    "→ Processing device",
    extra={
        "device_id": device_id,
        "device_ip": device_ip,
    },
)

## State changes (structured logging)
logger.info(
    "Device state changed",
    extra={
        "device_id": device_id,
        "old_state": old_state,
        "new_state": new_state,
    },
)

## Success (simple message)
logger.info("✓ Command sent successfully")

## Success with context (structured logging)
logger.info(
    "✓ Command sent successfully",
    extra={"device_id": device_id},
)

## Error with context (structured logging)
logger.error(
    "✗ Failed to send command",
    extra={
        "device_id": device_id,
        "error": error_msg,
    },
)

## Exception logging (structured logging)
logger.exception(
    "✗ Failed to send command",
    extra={
        "device_id": device_id,
        "error": str(e),
        "error_type": type(e).__name__,
    },
)

## Debug for protocol details (structured logging)
logger.debug(
    "Received packet",
    extra={"packet_hex": packet.hex(), "packet_length": len(packet)},
)

## Formatted display values (include in both message and extra={})
logger.debug(
    "Queued packet type 0x%02x",
    packet.packet_type,
    extra={"packet_type": packet.packet_type},
)
```

## Anti-Patterns

```python
## ❌ Don't log sensitive data
logger.info("Password: %s", password)

## ❌ Don't log in tight loops
for device in devices:
    logger.debug(f"Processing {device}")  # Too verbose

## ✅ Log summary instead
logger.debug(f"Processed {len(devices)} devices")
```

## Related

- See `logging-mandatory.mdc` for setup and when to log
- See `logging-patterns.mdc` for configuration patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jslamartina) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
