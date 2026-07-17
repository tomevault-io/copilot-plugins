## security

> |

# Security Implementation Rules

## Attack Safety
- **Rate Limiting**: Implement configurable rate limits for all attacks
- **User Consent**: Require explicit confirmation before executing attacks
- **Target Validation**: Validate target addresses and networks before attacks
- **Resource Limits**: Prevent resource exhaustion attacks
- **Timeout Mechanisms**: Implement timeouts for all network operations

## Privacy Protection
- **Data Minimization**: Only collect necessary information
- **Secure Logging**: Log sensitive operations without exposing details
- **Network Respect**: Respect network boundaries and policies
- **Audit Trails**: Maintain comprehensive logs for all operations
- **Data Cleanup**: Securely dispose of collected data

## Warning Systems
- **Clear Disclaimers**: Prominent warnings about legal and ethical use
- **Educational Purpose**: Emphasize educational/testing nature
- **Legal Notice**: Include legal disclaimers in documentation
- **Safety Guidelines**: Provide safety guidelines for users

## Project-Specific Security
- **WiFi Attacks**: All attacks must be rate-limited and safe
- **Platform Detection**: Robust OS and capability detection
- **Network Scanning**: Respect privacy and network policies
- **Audit Logging**: Comprehensive logging for all operations
- **CLI Warnings**: Clear warnings about legal and ethical use

---
> Source: [oyi77/wifi-jammer](https://github.com/oyi77/wifi-jammer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
