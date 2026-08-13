## wellbuilt-jsa

> Expo React Native **managed project** (SDK 54) for electronic Job Safety Analysis (JSA) forms. Part of the **WellBuilt Suite** — an oilfield services platform targeting water and oil hauling companies in the Williston Basin / Bakken. Built for Liquid Gold Trucking LLC in Williston, ND as first customer. Android package name: `com.syconik801.jsaapp`.

# CLAUDE.md - WB JSA (Electronic Job Safety Analysis)

## Project Overview
Expo React Native **managed project** (SDK 54) for electronic Job Safety Analysis (JSA) forms. Part of the **WellBuilt Suite** — an oilfield services platform targeting water and oil hauling companies in the Williston Basin / Bakken. Built for Liquid Gold Trucking LLC in Williston, ND as first customer. Android package name: `com.syconik801.jsaapp`.

**Sister app to WB Tickets** (`C:\dev\waterticket-app`). Both share the same Firebase project (`wellbuilt-sync`) and NDIC well data.

## Tech Stack
- **Framework:** Expo SDK 54, React Native 0.81.5, React 19.1.0
- **Language:** TypeScript 5.9.2 (strict mode)
- **Navigation:** Expo Router 6.0.15 (file-based routing in `app/` folder)
- **Backend:** Firebase (Firestore) - project `wellbuilt-sync` (shared with WB Tickets)
- **Build:** EAS Build (eas.json), New Architecture enabled
- **JS Engine:** Hermes (default)
- **Well Data:** NDIC wells from Firestore (same collections as WB Tickets — `operators`, `wells`, `operatorAliases`)
- **Animations:** React Native Reanimated 4.1.1

## Project Structure
```
C:\dev\JSA\
  app/                          # Expo Router pages (file-based routing)
    (tabs)/
      index.tsx                 # HOME: Driver/truck/date, NDIC well autocomplete, job type, pusher, lease/pad
      history.tsx               # HISTORY: List of saved/completed JSAs
      _layout.tsx               # Tab navigation layout (Job Details + Saved JSAs)
    _layout.tsx                 # Root layout, theme provider, language context
    steps.tsx                   # STEPS & HAZARDS: 9 sequential JSA steps with hazards/controls
    ppe.tsx                     # PPE CHECKLIST: Personal protective equipment selection (9 standard + 4 custom)
    signoff.tsx                 # SIGN OFF: Prepared checklist, notes, signature, submit
    pdf.tsx                     # PDF GENERATION: Printable JSA report via expo-print
    completed.tsx               # Completion confirmation screen
    openJsas.tsx                # Resume/view incomplete JSAs
    viewJsa.tsx                 # View details of a completed JSA
    modal.tsx                   # Generic modal screen
    contexts/
      LanguageContext.tsx        # i18n context (English/Spanish) — WARNING: Expo Router logs a warning about missing default export, harmless
  components/
    jsa/
      InputField.tsx            # Reusable input field component
      PrimaryButton.tsx         # Reusable button component
      SummaryCard.tsx           # Card showing JSA summary
  services/
    firebase.ts                 # Firebase init + Firestore CRUD for JSAs (connected to wellbuilt-sync)
    sync.ts                     # Cloud sync logic (device ID, upload/download)
    wellData.ts                 # NDIC well data access — loads operators/wells/aliases from Firestore, search, 24hr AsyncStorage cache
  constants/
    jsaTemplate.ts              # JSA hazards, controls, PPE items, emergency contacts, Loading/Unloading step data
    colors.ts                   # Color palette (gold primary #F5A623) & spacing system
    storageKeys.ts              # AsyncStorage key constants
    theme.ts                    # Theme configuration
  hooks/
    use-color-scheme.ts         # Detect system dark/light mode
    use-theme-color.ts          # Theme color helper
  types/
    jsa.ts                      # TypeScript type definitions
  assets/
    images/
      company-logo-transparent.png  # Liquid Gold Trucking logo
```

## Key Architecture Decisions
- **Managed Expo project** (NOT bare like WB Tickets — no native modules, no android/ folder)
- **Expo Router** for file-based routing (different from WB Tickets which uses React Navigation stack)
- **Shared Firebase project:** Same `wellbuilt-sync` Firestore as WB Tickets. Reads from `operators`, `wells`, `operatorAliases` collections.
- **JSA template hardcoded:** Loading/Unloading steps, hazards, controls, PPE, and emergency contacts are in `constants/jsaTemplate.ts` for Liquid Gold. Future: configurable per company via Firestore `companies/{id}/jsaTemplate`.
- **NDIC well autocomplete:** `services/wellData.ts` loads all ~19k wells from Firestore on app start, cached 24hr in AsyncStorage. Search requires 3+ characters. Shows well name, operator, county.
- **Offline-first:** JSAs saved to AsyncStorage, synced to Firestore when connected.
- **i18n:** Custom lightweight system (no library), English/Spanish toggle, ~120+ translated strings.
- **Light theme:** White background (#F5F5F5), gold accents (#F5A623), dark text (#111111). Different from WB Tickets' dark theme.

## App Flow
1. **Home** → Enter driver name, truck #, date, job type (Loading/Unloading), pusher, well(s) via NDIC autocomplete, optional lease/pad name
2. **Steps & Hazards** → Review 9 safety steps with hazards and controls, mark each complete
3. **PPE Checklist** → Select required PPE items (9 standard + 4 customizable "Other")
4. **Sign Off** → Confirm prepared-for-work checklist, add notes, sign (typed name), submit
5. **PDF** → Generate printable JSA report, share/export
6. **History** → View past JSAs, tap to see full details

## NDIC Well Data (shared with WB Tickets)
- **Collections:** `operators` (~103 docs), `wells` (~19,276 docs), `operatorAliases`
- **Client flow:** App starts → `loadOperators()` + `loadAliases()` + `loadAllWells()` (parallel) → cached in AsyncStorage (24hr TTL)
- **Search:** Driver types 3+ chars in Well Name field → `searchWells()` filters all wells by name → shows dropdown with well name, operator, county
- **Manual fallback:** If well not in NDIC, driver can type custom name and add manually
- **KNOWN ISSUE:** AsyncStorage SQLITE_FULL error when caching all ~19k wells. Wells still load from Firestore, just can't cache locally. Fix needed: chunked storage or compress cache.

## JSA Template Data (constants/jsaTemplate.ts)
- **9 JSA steps** for Loading task (generic): Driving on location, Backing, Connecting hoses, Loading fluids, Checking levels, Offloading fluids, Disconnecting hoses, Spill cleanup, Verify complete
- **Separate Loading/Unloading JSA records** with detailed step/hazard/control wording from Liquid Gold's actual paper JSA
- **PPE items:** Safety Glasses, Safety Shoes, FR Clothing, Hearing Protection, Hard Hat, Respirator, Chemical/Impact Gloves, Four Gas Monitor, Fall Protection + 4 customizable "Other"
- **Emergency contacts:** Travis Johnson, Phillip Nowak, Liquid Gold Dispatch, 911, ND Highway Patrol, Williams County Sheriff, McKenzie County Sheriff
- **Company contacts:** Dispatch, Nile LeBaron

## Build & Deploy
```bash
# Local development (Expo Go compatible — no native modules)
npx expo start

# EAS build (Android APK for testing)
eas build --profile preview --platform android

# EAS build (iOS)
eas build --profile preview --platform ios
```

## Known Issues & Current Status

### NDIC Well Autocomplete (2026-02-15) — WORKING ✅
- Well Name field uses NDIC autocomplete from shared Firestore data
- Search shows well name, operator, county in dropdown
- Multiple wells can be added to a single JSA
- Manual entry fallback if well not in NDIC
- AsyncStorage SQLITE_FULL warning when caching ~19k wells — cosmetic, data still loads from Firestore

### Firebase Connected (2026-02-15) — WORKING ✅
- Pointed at `wellbuilt-sync` project (same as WB Tickets)
- JSAs stored in `jsas` collection
- NDIC data read from shared `operators`, `wells`, `operatorAliases` collections

### Expo Router Warnings (harmless)
- `LanguageContext.tsx` missing default export warning — this is a context file, not a route. Harmless.
- `SafeAreaView` deprecation warning — should migrate to `react-native-safe-area-context`

### GitHub Repo
- **Repo:** https://github.com/SyCoNik99/JSA
- **Owner:** SyCoNik99 (daughter's account)
- **Local clone:** `C:\dev\JSA`
- **Collaborator:** WellBuilt (testerxxx@comcast.net)

### Future Improvements
- **Fix AsyncStorage SQLITE_FULL:** Chunk well data cache or use compressed format
- **Company-configurable JSA templates:** Move JSA steps/hazards/PPE/contacts from hardcoded `jsaTemplate.ts` to Firestore `companies/{id}/jsaTemplate`. AI scan-to-import: upload photo of paper JSA → Claude Vision API → structured JSON → Firestore.
- **SafeAreaView migration:** Replace deprecated `SafeAreaView` with `react-native-safe-area-context`
- **WellBuilt branding:** Update app name, logo, colors to match WB Suite
- **Signature:** Add drawn signature (like WB Tickets) instead of typed name
- **GPS capture:** Capture GPS location when JSA is submitted (prove driver was on-site)
- **Link JSA to invoice:** Associate JSA with WB Tickets invoice for the same job/day

## Code Conventions
- Light theme UI: background #F5F5F5, primary #F5A623 (gold), dark text #111111
- Portrait-only orientation
- File-based routing (Expo Router) — screens defined by file location in `app/`
- AsyncStorage keys defined in `constants/storageKeys.ts`
- Translations via `useLanguage()` hook → `t("key")` function
- Card-based layout with shadows (elevation 2)

## Gotchas
- **Managed project** — no `android/` or `ios/` folders. All native config via `app.json` and EAS.
- **Expo Go compatible** — can test on device without building APK (unlike WB Tickets which needs native modules)
- **Shared Firebase** — changes to Firestore rules/indexes for WB Tickets affect this app too
- **Two Metro bundlers** — if running both apps simultaneously, use different ports (`--port 8082` for JSA)
- **AsyncStorage size limit** — ~19k well records (~6MB JSON) can exceed SQLite storage on some devices. Need chunked storage solution.
- **Git remote** — pushes go to daughter's GitHub account (SyCoNik99/JSA). Coordinate changes to avoid merge conflicts.

---
> Source: [tester3x/wellbuilt-jsa](https://github.com/tester3x/wellbuilt-jsa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
