## clawcontrol

> - **Server Features**: The `ServerSettingsView.tsx` now includes a `FeaturesTab` supporting Text-to-Speech (TTS) and VoiceWake toggles, pulling data through the `openclaw/features.ts` API wrappers (`getTtsStatus`, `setTtsEnable`, etc.).

# OpenClaw Architecture & Context

## Recent Additions (v1.2+)
- **Server Features**: The `ServerSettingsView.tsx` now includes a `FeaturesTab` supporting Text-to-Speech (TTS) and VoiceWake toggles, pulling data through the `openclaw/features.ts` API wrappers (`getTtsStatus`, `setTtsEnable`, etc.).
- **Usage Insights**: Added a `UsageView.tsx` (accessible from the Sidebar) which loads data from `usage.status` and `usage.cost` displaying server load and cost estimations.
- **Cron Management**: `CronJobDetailView.tsx` was expanded to support `runCronJob` and `removeCronJob`. A new `CreateCronJobView.tsx` was built allowing full scheduling syntax (`cron.add`).
- **UI Layout**: Across all Detail views (`AgentDetailView`, `SkillDetailView`, `CronJobDetailView`, `ServerSettingsView`, `UsageView`, `CreateCronJobView`), a standard `.detail-header` wrapper was implemented for a uniform floating back-button and title lock-up.

## Navigation & UI State
- Zustand store (`store/index.ts`) manages active tabs through `mainView` (types updated to accommodate `usage` and `create-cron`).
- Features module added (`lib/openclaw/features.ts`) and fully exported through `openclaw/index.ts` to expose to the Electron/React client.

---
> Source: [jakeledwards/ClawControl](https://github.com/jakeledwards/ClawControl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
