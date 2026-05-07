## bill-tracker

> - This is an Expo application.

# Project: Expo React Native TypeScript Supabase Application

## General

- This is an Expo application.
- Use expo and bun for package manager.
- Use NativeWind CSS for styling, prefer to use color pink-400 and sky-400, should consider dark mode and use text-foreground and bg-background.
- Use sonner for toast.
- Use Zustand for state management.
- Use react-query for data fetching.
- Use ~/components for path alias.
- Use lucide-react-native for icons
- Use Lingui for i18n and prefer t`` macro
    * import { Trans } from "@lingui/react/macro"
    * import { t } from "@lingui/core/macro"
- Use date-fns for date formatting.
- Use lodash for utility functions.
- Component names are in CamelCase.

## File Structure for bibigpt-mobile

- app/(tabs): Main tabs.
- hooks/: React hooks.
- store/: Zustand store.
- components/: Components.
- components/ui: Pure UI Components.
- db/supabase/: Supabase db logic.
- utils/: Utils.
- sometimes, should save technology notes in /notes folder with Markdown in Chinese
  - only write the note when necessary, e.g. implements big feature or fix hard bug
  - should use bash script to get current date
  - the filename should start with date format yyyy-MM-dd
  - related notes should be grouped together in the one file

---
> Source: [JimmyLv/bill-tracker](https://github.com/JimmyLv/bill-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
