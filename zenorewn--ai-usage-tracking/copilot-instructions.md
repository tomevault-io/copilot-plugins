## ai-usage-tracking

> - Follow Kent Beck's TDD and Tidy First: behavioral tests before implementation, structural changes separately.

# Usage Tracking

Author: Zeno Ren

- Follow Kent Beck's TDD and Tidy First: behavioral tests before implementation, structural changes separately.
- Native macOS app; no mobile layout requirements.
- Keep usage data local. Read source transcripts and databases only; never persist prompts, responses, credentials, or tool arguments.
- Never silently sum unknown metrics, cache subsets, parent/child usage, or account quotas with local events.
- Each supported source must report coverage, precision, and failure states accurately.
- Preserve existing tool configuration; connection changes must be reversible and reviewable.
- Add Author: Zeno Ren to product About and generated artifacts; preserve third-party attribution.
- Run swift test and Scripts/build-app.sh before delivering application changes. Verify the actual app launches.

---
> Source: [ZenoRewn/AI-Usage-Tracking](https://github.com/ZenoRewn/AI-Usage-Tracking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
