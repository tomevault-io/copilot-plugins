## kindle-rewriter

> Repurpose ~50 donated Amazon Fire HD 7 tablets into minimal reading devices for kids in need. Strip Amazon bloatware, flash a custom LineageOS-based ROM with only a book reader (KOReader), a web browser, and a file manager. Pre-load with free public domain books.

# KindleRewriter - Agent Context

## Project Purpose

Repurpose ~50 donated Amazon Fire HD 7 tablets into minimal reading devices for kids in need. Strip Amazon bloatware, flash a custom LineageOS-based ROM with only a book reader (KOReader), a web browser, and a file manager. Pre-load with free public domain books.

## Two Target Devices

| | ariel (SQ46CW) | soho (P48WVB4) |
|---|---|---|
| **Device** | Fire HD 7 4th Gen (2014) | Kindle Fire HD 7 3rd Gen (2013) |
| **SoC** | MediaTek MT8135 (ARMv7) | TI OMAP 4470 (ARMv7) |
| **ROM base** | LineageOS 14.1 (Android 7.1.2) | LineageOS 11.0 (Android 4.4) |
| **Bootloader unlock** | amonet exploit (MT8135 branch) | fastboot oem unlock |
| **Device tree source** | github.com/amazon-oss/android_device_amazon_ariel | Adapted from LineageOS tate (OMAP4460) tree |
| **Maturity** | Good. Successfully built by r0rt1z2 in 2024 | Risky. No dedicated device tree; needs manual adaptation |

## Current State

**DONE:**
- Project scaffold with all scripts, configs, launcher app, and docs
- KidsLauncher Android app (Kotlin, zero dependencies, minSdk 19)
- LineageOS overlay configs (strip list + include list)
- Build scripts for both devices (setup, sync, build)
- Batch flash pipeline with USB watch mode for 50+ tablets
- Book fetcher (40 curated Project Gutenberg classics)
- Book loader (ADB push to one or all connected devices)
- udev rules and systemd service for auto-detection
- Full documentation (build guides for both devices, flashing guide for volunteers)

**TODO (on the build server):**
1. Run `setup-build-env.sh` to install deps and init repos
2. Run `sync-ariel.sh` to download LineageOS 14.1 source (~30GB)
3. Run `sync-soho.sh` to download LineageOS 11.0 source (~20GB)
4. Verify ariel device tree repos exist at amazon-oss org on GitHub. If the repos are gone or broken, check XDA for mirrors: https://xdaforums.com/t/rom-unofficial-7-1-2-lineageos-14-1-for-amazon-fire-hd6-7-ariel.4702111/
5. Create the soho device tree by copying and adapting the tate tree (see docs/soho-build-guide.md)
6. Build both ROMs
7. Download TWRP recovery images for both devices
8. Download KOReader APK and place in overlay/vendor/kindle-rewriter/prebuilt/KOReader.apk
9. Run `fetch-gutenberg.sh` to download the book collection
10. Test flash on one tablet of each type before batch processing

## Critical Gotchas

- **ariel bootloader unlock requires amonet, NOT fastboot.** The tablet must be in MediaTek download mode (Vol Down + Power), not fastboot mode. The amonet repo branch is `mt8135-ariel`.
- **soho has no dedicated device tree.** You must adapt the `tate` tree. Key differences: OMAP4470 vs OMAP4460 (minor clock/GPU diffs), different firmware blobs. Extract proprietary blobs from a stock soho tablet via ADB.
- **soho is the risky build.** If it proves too difficult, the fallback is: flash the tate ROM (same SoC family) and sideload KidsLauncher + KOReader APKs manually.
- **MediaTek USB issues:** Use USB 2.0 ports, not USB 3.0, for amonet. Add udev rules first (`install-udev-rules.sh`).
- **LineageOS 14.1 uses the `cm-14.1` branch name**, not `lineage-14.1`. Same for cm-11.0.
- **No GApps, ever.** The entire point is no Play Store, no YouTube, no Google services. The config.mk strip list enforces this but double-check the build output.
- **KidsLauncher targets minSdk 19** to be compatible with both Android 4.4 (soho) and 7.1.2 (ariel). It uses zero AndroidX dependencies, only framework APIs.

## Build Requirements

- Ubuntu 18.04+ (20.04 works)
- 350GB+ free disk space (source + build artifacts for both devices)
- 16GB+ RAM (8GB works with swap but is slow)
- Java 8 or 11
- Python 3 (for amonet)

## Key External Repos

- amonet (ariel unlock): https://github.com/R0rt1z2/amonet/tree/mt8135-ariel
- r0rt1z2's ariel writeup: https://blog.r0rt1z2.com/posts/hacking-2014-tablet/
- KOReader: https://github.com/koreader/koreader/releases
- LineageOS 14.1 ariel thread: https://xdaforums.com/t/rom-unofficial-7-1-2-lineageos-14-1-for-amazon-fire-hd6-7-ariel.4702111/
- LineageOS omap4-common: https://github.com/LineageOS/android_device_amazon_omap4-common
- postmarketOS soho page: https://wiki.postmarketos.org/wiki/Amazon_Kindle_Fire_HD_3rd_Generation_(amazon-soho)

## Design Decisions

- **No custom reader app (yet).** We bundle KOReader because it's battle-tested and supports every ebook format. A kid-friendly custom reader could be a phase 2 project.
- **Dark theme on launcher.** Chosen for battery life on LCD and to look distinct from stock Android. Colors: #1a1a2e background, #e94560 accent.
- **Browser fallback chain.** KidsLauncher tries: Nico Browser, Jelly (LineageOS default), stock Android browser, Firefox, Opera Mini. At least one should be available.
- **Batch flash tracks state per device serial.** If a tablet is unplugged mid-process, plugging it back in resumes from the last completed stage.

## Commit Style

- No AI attribution in commits or PRs
- Keep messages clean and descriptive
- Use imperative mood ("Add feature" not "Added feature")

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kadajett)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kadajett)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
