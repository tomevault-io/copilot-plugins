## gdpr-privacy

> GDPR — privacy policy version, consent, IP hashing


# GDPR & Privacy Policy

## IP & logs
Security/rate-limit logs: IP as SHA256 + `ADV_THROTTLE_SALT` only. No plaintext personal data in logs.

## Substantial change to `resources/views/legal/privacy.blade.php`
Complete in **same** PR/session:
1. Update visible version/date (e.g. `updated-at` on `<x-legal-page-shell>`).
2. Bump `privacy_policy_version` in `config/legal.php` (unique string, e.g. `YYYY-MM-DD-v2`).
3. Consent controllers use **only** `config('legal.privacy_policy_version')` — no duplicated version strings.
4. Update UI/tests that cite policy version or binding text.
5. `make test` before done.

Version feeds `consents` / `consent_events`. Legal-only edits without version bump are **incomplete**.

---
> Source: [mnossa/finanzamente](https://github.com/mnossa/finanzamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
