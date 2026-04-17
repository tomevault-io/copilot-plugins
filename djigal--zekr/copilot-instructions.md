## zekr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zekr is an open-source, cross-platform Quran study application built with Java and SWT. It provides Quranic text display, translations, search capabilities, audio recitation, and bookmarking features with support for 17+ languages.

## Build System

**Build Tool:** Apache Ant

**Common Commands:**
```bash
# Build the project (extracts source from dist/zekr-src.jar, compiles, and creates dist/zekr.jar)
ant dist

# Compile only
ant compile

# Generate JavaDoc
ant javadoc

# Run the application
ant run

# Clean build artifacts
ant clean
```

**Important:** The source code is distributed as `dist/zekr-src.jar` and is extracted to `build/src/` during the build process. Compiled classes are placed in `build/classes/`, and the final JAR is created at `dist/zekr.jar`.

## Running the Application

**Linux/Unix:**
```bash
./zekr.sh
```

**Direct Java execution:**
```bash
java -Xms20m -Xmx128m -cp "lib/*:dist/zekr.jar" net.sf.zekr.ZekrMain
```

## Architecture

### Technology Stack

- **GUI Framework:** SWT (Standard Widget Toolkit) - Native cross-platform UI
- **Template Engine:** Apache Velocity 1.6.2 - HTML generation for Quran display
- **Search Engine:** Apache Lucene 3.0.0 - Full-text indexing and multi-language search
- **Audio Playback:** JLayer, Tritonus, JOrbis, JSpeex - Multi-format audio support
- **Logging:** Apache Log4j 1.2.8
- **HTTP Server:** Embedded NanoHTTPD on port 8920
- **Target:** Java 1.5

### Package Structure

The main package is `net.sf.zekr` with key subpackages:

- **`net.sf.zekr.ZekrMain`** - Application entry point
- **`net.sf.zekr.engine.search`** - Lucene-based search with multi-language stemming and query parsing
- **`net.sf.zekr.engine.audio`** - Audio playback engine for online/offline recitations with caching
- **`net.sf.zekr.engine.template`** - Velocity template rendering with custom resource loaders
- **`net.sf.zekr.engine.i18n`** - Internationalization and multi-language support
- **`net.sf.zekr.engine.data`** - Quranic data structures (Sura, Aya, revelation data)
- **`net.sf.zekr.engine.bookmark`** - XML-based bookmark management with import/export
- **`net.sf.zekr.ui`** - SWT-based user interface components
- **`net.sf.zekr.engine.config`** - Configuration management using Apache Commons Configuration
- **`net.sf.zekr.engine.log`** - Custom Log4j implementation

### Resource Directory Structure

**`res/`** - All application resources organized by type:

- **`config/`** - Application configuration files
  - `config.properties` - Main runtime settings (UI, audio, search, network)
  - `search-info.properties` - Multi-language search configuration (stop words, diacritics)
  - `shortcut.xml` - Keyboard shortcuts with RTL support
  - `lib/logger.properties` - Log4j configuration (rolling file appender, 400KB limit)
  - `lib/velocity.properties` - Velocity template engine settings

- **`text/`** - Quranic text and metadata
  - `quran/` - Quran text files (quran-simple.txt, quran-uthmani.txt from Tanzil)
  - `trans/` - Translation packages (.trans.zip files)
  - `metadata/` - Quran metadata in 15+ languages (sura names, properties)
  - `index/` - Lucene search indices (quran/, trans/)

- **`audio/`** - Recitation metadata
  - `.properties` files for each reciter with URL templates for streaming
  - Supports both online and offline playback

- **`ui/`** - UI themes and presentation
  - `theme/` - Multiple themes (sky/, uthman-taha/)
    - `*.vm` - Velocity templates (page.vm, search-result.vm, aya-result.vm)
    - `css/` - Stylesheets (common-style.css, screen-style.css, print-style.css)
    - `theme.properties` - Font, color, layout configuration
  - `js/` - JavaScript libraries (jQuery, custom scripts)

- **`lang/`** - Internationalization files
  - 17+ language packs in XML format (English, Arabic, Persian, Turkish, etc.)
  - Each includes UI strings and locale direction (LTR/RTL)

- **`bookmark/`** - Bookmark management
  - `bookmarks.xml` - Default bookmark collection
  - `template/` - Bookmark templates and XSLT export

- **`image/`** - Icons and graphics
  - Organized by function (icon/, icon/audio/, icon/bookmark/, icon/flag/)
  - Multiple sizes (16px, 32px, 48px, 128px, 256px variants)

## Key Configuration Files

**`config.properties`** - Main configuration with Velocity variable support:
- Quran layout and translation display settings
- Audio player behavior and caching (200MB capacity)
- Search configuration (max results, parsers, highlighting)
- Theme and language defaults
- Server and network settings

**`resource-path.properties`** - Dynamic path resolution:
- Uses `{1}` placeholder for user workspace directory
- Defines paths for images, configs, languages, themes, text, bookmarks

**`search-info.properties`** - Search engine configuration:
- Per-language character normalization rules
- Stop words for multiple languages
- Diacritic handling patterns
- Unicode letter range definitions

## Development Notes

### Audio System
- Supports multiple reciters with different bitrates (16kbps to 192kbps)
- Playback modes: single aya, multi-aya, page, sura, juz, hizb-quarter, continuous
- 200MB cache with prefetching for offline playback
- URL template format: `http://server/path/%1$03d%2$03d.mp3` (sura/aya)

### Search Engine
- Lucene 3.0 with custom analyzers per language
- Supports advanced queries: proximity, wildcard, fuzzy, boolean
- Query parsers: QueryParser, AnalyzingQueryParser, ComplexPhraseQueryParser
- Snowball stemming for multiple languages
- Result highlighting with customizable fragments

### Template System
- Velocity-based HTML generation for Quran display
- Custom resource loader (`ZekrFileResourceLoader`)
- Theme switching at runtime
- Supports mixed/separate translation layouts
- Print-specific CSS for proper formatting

### Internationalization
- XML-based language packs with UTF-8 encoding
- RTL (right-to-left) support for Arabic, Persian, Urdu, Hebrew
- Dynamic locale switching without restart
- Translatable metadata (sura names, revelation locations)

### Bookmark System
- XML storage with tree-based bookmark sets
- XSLT-based export to web format (HTML)
- Import/export functionality

### Build Process
1. Source is extracted from `dist/zekr-src.jar` to `build/src/`
2. Java source compiled with all dependencies to `build/classes/`
3. `.properties` files copied as resources
4. Final executable JAR created at `dist/zekr.jar`
5. Dependencies loaded from `lib/` directory at runtime

## Dependencies

All JARs are located in the `lib/` directory:

**Core Libraries:**
- swt.jar - SWT GUI toolkit (platform-specific)
- velocity-1.6.2.jar - Template engine
- log4j-1.2.8.jar - Logging framework

**Lucene Search:**
- lucene-core-3.0.0.jar, lucene-highlighter-3.0.0.jar
- lucene-snowball-3.0.0.jar, lucene-memory-3.0.0.jar, lucene-misc-3.0.0.jar

**Apache Commons:**
- commons-lang-2.4.jar, commons-io-1.4.jar, commons-collections-3.2.1.jar
- commons-logging-1.0.4.jar, commons-codec-1.3.jar, commons-configuration-1.6.jar

**Audio:**
- jlayer-1.0.1.jar, basicplayer-3.0.jar, mp3spi-1.9.4.jar
- tritonus-share-0.3.6.jar, tritonus-jorbis-0.3.6.jar
- jorbis-0.0.17.jar, jspeex-0.9.7.jar, vorbisspi-1.0.3.jar

**Platform-Specific:**
- jintellitype-1.3.8.jar - Windows global hotkeys

## Extensibility Points

- **Audio Sources:** Add new reciters via `.properties` files in `res/audio/`
- **Themes:** Create new themes in `res/ui/theme/` with Velocity templates and CSS
- **Languages:** Add language packs as XML files in `res/lang/`
- **Translations:** Package translations as `.trans.zip` files in `res/text/trans/`
- **Search Scope:** Configure custom Lucene query builders in search-info.properties

## JVM Configuration

- Minimum heap: 20MB (`-Xms20m`)
- Maximum heap: 128MB (`-Xmx128m`)
- Native library path: `lib/` directory for SWT native bindings
- Optional browser configuration: `-Dorg.eclipse.swt.browser.XULRunnerPath` for custom Firefox/XULRunner

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/djigal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
