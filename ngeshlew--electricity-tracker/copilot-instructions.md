## electricity-tracker

> - Identify candidate Web APIs (Streams/TransformStream, WebAuthn/passkeys, Service Workers/Manifest, File System Access, Web Share, Badging, Background Sync, Shortcuts, URL handlers)

# web.dev Best Practices Overlay for RIPER Σ
# Version: 1.0.0

- Mode overlays
  - Research (Ω₁)
    - Identify candidate Web APIs (Streams/TransformStream, WebAuthn/passkeys, Service Workers/Manifest, File System Access, Web Share, Badging, Background Sync, Shortcuts, URL handlers)
    - Define progressive enhancement and fallbacks for each feature
    - Set performance targets (p75): LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1; optional TTFB ≤ 800 ms
  - Plan (Ω₃)
    - Specify PWA strategy (offline, install, update, navigation) and manifest requirements (icons, display, shortcuts, scope)
    - Define passkey flows (registration, sign‑in) with fallback; security posture (`userVerification: required`, `attestation: none`)
    - Specify streaming/data pipeline and backpressure handling
    - Define RUM schema, sample rate (1–10%), and endpoint
  - Execute (Ω₄)
    - Implement progressive enhancement with feature detection
    - Implement service worker caching + offline fallback; manifest with app‑like features
    - Use Streams API for large data; provide non‑stream fallback
    - Implement passkeys with WebAuthn; rotate challenges; verify RP ID and origin
    - Integrate `web-vitals` and send metrics (LCP/INP/CLS) with attribution
  - Review/Test (Ω₄/Ω₅)
    - Verify field p75 metrics meet budgets (LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1)
    - Verify offline/install/auth flows on two browsers and a mid‑tier device profile
    - Use attribution to localize regressions (LCP element, INP interaction, CLS sources)

- Performance Budgets (Κ₄ gate addendum)
  - performance_benchmarks_met = {
      LCP.p75_ms ≤ 2500,
      INP.p75_ms ≤ 200,
      CLS.p75 ≤ 0.1
    }

- Implementation Reminders
  - Use feature detection; do not block baseline UX when features are unavailable
  - Respect privacy for telemetry; avoid PII; keep payloads small; sample data
  - Prefer app‑like navigation patterns and stable layouts to avoid CLS

- References
  - See `docs/webdev-integration.md` for examples and checklists

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ngeshlew) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
