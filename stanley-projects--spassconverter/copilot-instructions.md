## spassconverter

> Android app that converts Samsung Pass (`.spass`) encrypted export files into a standard CSV format compatible with password managers like Bitwarden, 1Password, and others.

# SPASS Converter

Android app that converts Samsung Pass (`.spass`) encrypted export files into a standard CSV format compatible with password managers like Bitwarden, 1Password, and others.

- Package: `com.stanley.spassconverter`
- Min SDK: 26 (Android 8.0)
- Target SDK: 36
- Fully offline — no internet permissions, no tracking
- Jetpack Compose UI

## Build

```bash
./gradlew assembleRelease
```

Release signing requires `local.properties` to define:
```
RELEASE_STORE_FILE=<path to keystore>
RELEASE_STORE_PASSWORD=<password>
RELEASE_KEY_PASSWORD=<password>
```

Without those properties the build compiles and produces an unsigned release APK (suitable for F-Droid or CI).

## Distribution

- Google Play Console: uploaded, blocked on 12-tester closed-testing requirement.
- F-Droid: metadata prepared, submission pending.
- Sideload: install release APK directly on device.

Release keystore path and signing credentials are in `local.properties` (gitignored).

---

## Agent Memory Protocol

Use this file as the standing project memory shared between Claude Code and Codex.

Rules:

1. Any meaningful code change, architecture decision, bug fix, regression, or workflow adjustment should be recorded here.
2. Each note should be prefixed with the agent name in brackets, for example `[Claude]` or `[Codex]`.
3. Prefer concise entries that explain what changed, why it changed, and any follow-up risk.
4. Keep older entries, including failed attempts and mistakes. Do not rewrite agent log history; append corrections and follow-up outcomes instead.
5. If the file's encoding gets damaged, normalize it back to plain ASCII or valid UTF-8 while preserving meaning.

Suggested note format:

```text
### YYYY-MM-DD
- [AgentName] What changed. Why it changed. Any follow-up note.
```

## Agent Log

### 2026-04-25
- [Claude] Added Sam's Club package mapping (`com.rfi.sams.android`) to AppUrlMapper — fallback heuristic produced wrong domain (`rfi.com`).
- [Claude] Added UTF-8 BOM to CsvExporter output — spreadsheet tools displayed mojibake without it.
- [Claude] Refactored ConverterViewModel: `password` moved into ConversionState, added `clearOnUserLeave()` — prevents sensitive state persisting after user leaves app.
- [Claude] Added `skipClearOnNextUserLeaveHint` gate in MainActivity — `onUserLeaveHint` was firing during app-initiated picker launches, clearing `fullCsv` before write and producing empty saved files.
- [Claude] Bumped versionCode 1→2, versionName 1.0→1.0.1.

### 2026-05-03
- [Claude] Fixed build.gradle.kts: moved hardcoded keystore path into `local.properties` as `RELEASE_STORE_FILE`; made entire signing config conditional on that key being present. Enables clean builds on F-Droid servers and other machines without the keystore.
- [Claude] Added MIT LICENSE file — required for F-Droid listing.
- [Claude] Added fastlane metadata under `fastlane/metadata/android/en-US/` — F-Droid uses this for app store listing display.

### 2026-05-27
- [Claude] Moved `keyAlias` from build.gradle.kts into `local.properties` as `RELEASE_KEY_ALIAS`; added to `hasSigningConfig` guard — no signing info in any committed file now.
- [Claude] Cleaned CLAUDE.md: removed machine-specific keystore path and sideload APK path.
- [Claude] Full SEO/AEO pass: 20 GitHub topics via API, keyword-rich repo description, homepage set to Pages URL.
- [Claude] Added README.md (lead sentence, badges, feature list, output table, step-by-step guide, FAQ, keyword footer) and CITATION.cff.
- [Claude] Built full GitHub Pages site in docs/: index.html landing page (JSON-LD MobileApplication + FAQPage), samsung-pass-to-google.html (HowTo schema), open-spass-file.html (Article + FAQPage), faq.html (FAQPage with 16 questions), privacy.html, terms.html, favicon.svg, og-image.svg (1200×630).
- [Claude] AI/LLM files: llms.txt + llms-full.txt (llmstxt.org format with canonical answer block), robots.txt (explicitly allows 20+ AI crawlers + GPTBot, ClaudeBot, PerplexityBot, Google-Extended, etc.), sitemap.xml (github.io URLs only), humans.txt, .well-known/security.txt, .nojekyll.
- [Claude] Pushed; Pages live at https://stanley-projects.github.io/SpassConverter/ (HTTP 200 on first poll).
- [Claude] IndexNow submitted 4 URLs to api.indexnow.org — HTTP 202 Accepted.
- [Claude] Social preview PNG rendered via headless Edge → C:\Users\HP\Downloads\SpassConverter-social-preview.png (161 KB).

---
> Source: [stanley-projects/SpassConverter](https://github.com/stanley-projects/SpassConverter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
