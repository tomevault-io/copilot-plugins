## career-compass

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Career Compass is a privacy-first career exploration desktop application built as a hybrid Next.js/Electron app. It analyzes resumes locally using multiple LLM providers to suggest personalized career paths. The architecture prioritizes user privacy with local file processing and secure storage.

## Common Commands

### Development
```bash
npm run dev                    # Start Next.js development server
npm run electron:dev           # Start both Next.js and Electron (main development command)
npm run electron:dev-unsafe    # Development with disabled web security for debugging
```

### Building
```bash
npm run build                  # Build Next.js static export
npm run electron:pack          # Package Electron app for testing (no installer)
npm run electron:dist          # Build production distributables for current platform
```

### Release Process
```bash
# Update version in package.json, commit, then:
git tag v0.x.x
git push origin v0.x.x        # Triggers GitHub Actions multi-platform build
```

## Architecture Overview

### Hybrid Web/Desktop Application
- **Next.js 14** with App Router and static export (`output: 'export'`)
- **Electron wrapper** provides desktop functionality while maintaining web compatibility
- **Dual runtime environments**: Web (localStorage) vs Desktop (electron-store + secure storage)

### Multi-Provider LLM Architecture
Located in `lib/llm-providers.ts`, implements a provider abstraction pattern:
- **Common interface** (`LLMProvider`) for all AI providers
- **OpenAI SDK** used as base client for API compatibility across providers
- **Environment variable fallback** when no stored API keys
- **Connection testing** with provider-specific health check endpoints

Supported providers:
- **Ollama** (local, privacy-first default)
- **OpenAI** (GPT models)
- **Anthropic Claude**
- **Groq** (fast inference)
- **Google Gemini**

### Settings and Storage Architecture
Dual storage system in `lib/settings-store.ts`:

**Desktop (Electron)**:
- Settings: `electron-store` in user data directory
- API Keys: `safeStorage` with OS-native encryption (keychain/credential manager)
- **Async IPC operations** - all store methods return Promises

**Web (Fallback)**:
- Settings: `localStorage`
- API Keys: `localStorage` (less secure)

**Critical Implementation Detail**: Settings store operations are async to handle IPC communication. Always use `await` when calling `settingsStore.get()`, `settingsStore.set()`, etc.

### File Processing Pipeline
Located in `lib/file-processors.ts` with API routes in `app/api/parsePdf/`:
- **PDF**: `pdf-parse` library (configured as external package in Next.js)
- **DOCX**: `mammoth` library for Word document conversion
- **Markdown**: Direct text processing
- **Client-side validation** before server processing
- **ArrayBuffer handling** for binary file data

### Security Architecture

**Electron Security Hardening**:
- `contextIsolation: true` with secure IPC bridge (`electron/preload.js`)
- `nodeIntegration: false` and `webSecurity: true`
- External URLs opened in system browser, not within app
- Secure API key storage using OS-native encryption

**Privacy Design**:
- Local file processing (files never sent to external servers except LLM APIs for analysis)
- No analytics or tracking implementation
- User data remains on device

### IPC Communication Pattern
Between main and renderer processes via `electron/preload.js`:
```javascript
// Settings operations
window.electronAPI.store.get(key, defaultValue)
window.electronAPI.store.set(key, value)

// Secure storage for API keys
window.electronAPI.secureStorage.setPassword(service, password)
window.electronAPI.secureStorage.getPassword(service)

// Model management
window.electronAPI.models.getOllamaModels(baseURL)
window.electronAPI.models.testConnection(provider, config)
```

### Career Generation Workflow
Two-stage LLM process in `app/api/getCareers/route.ts`:
1. **Initial Analysis**: Resume + context → 6 career suggestions with basic info
2. **Detailed Analysis**: Each career → comprehensive roadmap, skills analysis, timeline

**ReactFlow Integration**: Career suggestions rendered as interactive nodes in `components/CareerNode.tsx` with visualization in `app/careers/page.tsx`.

## Build Configuration

### Next.js Configuration (`next.config.mjs`)
```javascript
output: 'export'                                    // Static export for Electron
serverComponentsExternalPackages: ['pdf-parse']     // External package handling
images: { unoptimized: true }                       // Required for static export
```

### Electron Builder (`package.json` build section)
- **Cross-platform targets**: Windows (NSIS), macOS (DMG), Linux (AppImage)
- **Artifact naming**: `Career-Compass-{version}-{arch}.{ext}` (no spaces)
- **Auto-updater**: GitHub releases integration
- **Code signing**: Placeholder configuration for certificates

### GitHub Actions (`/.github/workflows/release.yml`)
- **Matrix build**: Parallel builds on macOS, Windows, Ubuntu
- **Tag-triggered**: Activates on `v*` tags (e.g., `v0.2.0`)
- **Automated releases**: Creates GitHub release with platform-specific binaries

## Development Guidelines

### Environment Variable Support
Settings system supports environment variables as fallback:
- `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GROQ_API_KEY`, `GOOGLE_API_KEY`
- Accessed via `window.electronAPI.getEnvVar()` in renderer process
- Check `lib/settings-store.ts` for precedence order

### Error Handling Patterns
- **Toast notifications** using `react-hot-toast` for user feedback
- **Graceful degradation** when desktop features unavailable in web mode
- **Fallback stores** when electron-store initialization fails

### File Structure Conventions
- `/app/*` - Next.js App Router pages and API routes
- `/lib/*` - Core business logic and utilities
- `/components/*` - React components (UI components in `/components/ui/`)
- `/electron/*` - Desktop-specific code (main process and preload script)

### Testing Connection to LLM Providers
Connection testing implementation varies by provider:
- **Ollama**: `GET /api/tags` endpoint
- **OpenAI/Groq**: `GET /v1/models` endpoint
- **Claude**: `POST /v1/messages` with minimal test request
- **Gemini**: `GET /v1beta/models` endpoint

All connection tests include API key validation and environment variable fallback logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michael-borck) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
