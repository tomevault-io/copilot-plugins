## 02-privacy

> Privacy-by-design rules for handling PII, logging, telemetry, and data minimization.


# Privacy Rules

## Data Minimization

- Only collect, store, and process data that is strictly necessary.
- Prefer anonymous or pseudonymous identifiers over PII.
- Set data retention limits — do not store data indefinitely.

## PII Handling

- Never log PII (names, emails, IP addresses, phone numbers) in application logs.
- Mask or redact PII in error messages and log output.
- Encrypt PII at rest and in transit.
- Mark PII fields in database schemas with clear comments.

## Telemetry & Analytics

- Never add tracking or analytics without consent mechanisms.
- All analytics must be opt-in or provide clear opt-out.
- Never send PII to third-party analytics services.

## Compliance

- Support GDPR rights: access, rectification, erasure, portability.
- Include consent mechanisms where personal data is collected.
- Follow the most restrictive applicable regulation.

---
> Source: [the-missing-pink/ai-repository-security-baseline](https://github.com/the-missing-pink/ai-repository-security-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
