## office-coverter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Office-Converter is a Spring Boot web application that converts various office file formats (HWP, HWPX, DOC, DOCX, XLS, XLSX, PPT, PPTX) to PDF format. The application uses two conversion engines:
- **Hancom SDK** (native libraries) for Korean HWP/HWPX files
- **JODConverter + LibreOffice** for other office formats (MS Office, OpenOffice, etc.)

**Important:** This application only runs on Linux due to native library dependencies.

## Build and Run Commands

### Build
```bash
./gradlew build
```

### Run locally
```bash
./gradlew bootRun
```

### Create WAR file
```bash
./gradlew war
```
The WAR file will be generated in `build/libs/`

### Clean build
```bash
./gradlew clean build
```

## System Requirements and Setup

### Prerequisites (Linux only)
1. Install LibreOffice:
   ```bash
   sudo add-apt-repository ppa:libreoffice/ppa
   sudo apt update
   sudo apt install libreoffice
   libreoffice --version
   ```

2. Set up native libraries:
   - Required native `.so` files must be present in the Java library path
   - `hnc/` directory must be present with Hancom SDK resources
   - `bin/` directory must be present (not in repo, uploaded separately)
   - See https://github.com/Ablestor/office-converter-lib

3. Configure Java library path for native libraries

**CRITICAL:** Do not change the package name (`com.hancom.weboffice`) as it's tied to the native library bindings.

## Architecture

### Conversion Flow

The application provides three REST API endpoints that handle different conversion scenarios:

1. **HWP files** (`/japi/hwp/convert-to-pdf`):
   - Uses Hancom SDK via JNI (WebhwpUnix.java)
   - Writes files to disk, converts, returns as Resource
   - Cleans up temporary files after conversion

2. **HWPX files** (`/japi/hwpx/convert-to-pdf`):
   - Same as HWP but specifies "HWPX" format type
   - Uses Hancom SDK via JNI

3. **Other office files** (`/japi/all/convert-to-pdf`):
   - Uses JODConverter with LibreOffice backend
   - Streams conversion in-memory (ByteArrayOutputStream)
   - Returns byte array directly without disk I/O

### Key Components

**RestApiController.java** (src/main/java/com/hancom/weboffice/controller/RestApiController.java:30)
- Main REST controller handling all conversion endpoints
- Initializes JODConverter with LocalOfficeManager on port 2002
- Manages temporary file cleanup for HWP conversions

**WebhwpUnix.java** (src/main/java/com/hancom/weboffice/hwp/filter/WebhwpUnix.java:8)
- JNI wrapper for Hancom native libraries
- Loads 6 native libraries: hsp-nox, HncXerCore, HncXalMesg, HncPtnCore, HncXalCore, HncXsecCore, hwp_sdk_unix
- Implements retry logic (3 attempts with 300ms delay) for conversion operations
- Configures HNC_DIR, HNC_SHARED_DIR, HNC_TEMP_DIR from `hnc/` directory

**HwpApplication.java** (src/main/java/com/hancom/weboffice/HwpApplication.java:9)
- Spring Boot application entry point
- Extends SpringBootServletInitializer for WAR deployment

### Configuration

**application.properties**:
- Multipart file upload: max 100MB
- JSP views: `/WEB-INF/views/*.jsp`
- Temp folder: `/tmp` (configurable via `tmp-folder` property)

**build.gradle**:
- Spring Boot 2.5.4
- Java 1.8 compatibility
- JODConverter 4.4.x dependencies
- Custom JARs in `libs/` directory (not in repo)

## Development Notes

### Native Library Integration
- WebhwpUnix static initializer loads all native libraries at class load time
- If libraries fail to load, application prints error but continues (conversions will fail)
- Native methods use JNI naming convention and are implemented in `.so` files

### File Handling
- HWP/HWPX conversions write temporary files to `tmp-folder` with UUID names
- Temporary input files are deleted after conversion
- Temporary output PDF files remain at `/tmp/<uuid>.pdf` for response
- General office conversions use in-memory streams (no disk I/O)

### Error Handling
- All conversion endpoints return 500 Internal Server Error on failure
- Errors are logged but not exposed to client
- HWP SDK errors use HRESULT-style error codes (0x80000000 bit indicates failure)

### Port Configuration
- JODConverter LocalOfficeManager uses port 2002 for LibreOffice communication
- Ensure this port is available before starting the application

## Common Issues

1. **"Native code library failed to load"**: Missing or incorrect `.so` files in java.library.path
2. **HWP conversion fails**: Check `hnc/` directory structure (must have Shared/, temp/, upload/ subdirectories)
3. **LibreOffice conversion fails**: Verify LibreOffice is installed and port 2002 is available
4. **File size limits**: Increase max-file-size/max-request-size in application.properties if needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/damob-byun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
