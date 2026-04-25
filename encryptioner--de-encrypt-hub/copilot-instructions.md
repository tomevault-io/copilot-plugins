## de-encrypt-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

De-encrypt Hub is a browser-based encryption and decryption toolkit built with React, TypeScript, and Vite. It provides secure, client-side cryptographic operations including symmetric/asymmetric encryption, hashing, digital signatures, and image encryption.

**Live Demo**: [https://encryptioner.github.io/de-encrypt-hub/](https://encryptioner.github.io/de-encrypt-hub/)

## Development Commands

```bash
# Install dependencies
pnpm i

# Start development server (runs on port 8080)
pnpm run dev

# Build for production
pnpm run build

# Build for development mode
pnpm run build:dev

# Run linting
pnpm run lint

# Preview production build
pnpm run preview
```

## Code Architecture

### Core Structure
- **Entry Point**: `src/main.tsx` → `src/App.tsx` → `src/pages/Index.tsx`
- **Component Architecture**: Single-page app with tabbed interface organized by encryption category
- **State Management**: React hooks with custom crypto-specific hooks in `src/hooks/`
- **Styling**: Tailwind CSS with shadcn/ui components

### Key Directories

- `src/components/`: Main UI components
  - `DeencryptHub.tsx`: Central tabbed interface component
  - `CipherTool.tsx`, `RsaEncryptionTool.tsx`, etc.: Individual crypto tools
  - `ui/`: shadcn/ui component library
- `src/hooks/`: Custom React hooks for crypto operations
  - `useCipher.ts`, `useRsa.ts`, `useHash.ts`: Crypto-specific logic
- `src/lib/`: Core utilities
  - `crypto.ts`: Symmetric encryption and hashing functions
  - `utils.ts`: General utilities and shadcn/ui helpers
- `src/config/content.json`: Algorithm descriptions and educational content

### Crypto Implementation Details

- **Symmetric Encryption**: Uses CryptoJS library (AES, DES, TripleDES, Rabbit, RC4)
- **Asymmetric Encryption**: Web Crypto API for RSA-OAEP operations
- **Digital Signatures**: RSA-PSS and Ed25519 implementations
- **Hashing**: Web Crypto API (SHA-1, SHA-256, SHA-512) + CryptoJS (MD5)
- **Image Encryption**: Custom pixel manipulation algorithms
- **JWT**: jose library for JSON Web Token operations

### Component Patterns

1. **Tool Components**: Follow pattern of mode prop ("encrypt" | "decrypt") for dual-purpose tools
2. **Hook Integration**: Each crypto operation has dedicated custom hook
3. **Error Handling**: Consistent error boundaries and user feedback via toast notifications
4. **Theme Support**: Dark/light mode via next-themes

### Important Configuration

- **Path Aliases**: `@/` maps to `src/` directory
- **Build Target**: ES2020 with modern browser support
- **Port**: Development server runs on port 8080
- **Package Manager**: Uses pnpm (lockfile: pnpm-lock.yaml)
- **Base Path**: Production builds use `/de-encrypt-hub/` for GitHub Pages compatibility

### Deployment

- **Platform**: GitHub Pages
- **Workflow**: `.github/workflows/deploy.yml`
- **Trigger**: Automatic deployment on push to `main` branch
- **Build Output**: `dist/` directory
- **Live URL**: [https://encryptioner.github.io/de-encrypt-hub/](https://encryptioner.github.io/de-encrypt-hub/)

### Security Considerations

All cryptographic operations run client-side in the browser. No data is transmitted to external servers. The application is designed for educational and utility purposes with proper security warnings for deprecated algorithms (MD5, SHA-1, DES).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Encryptioner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
