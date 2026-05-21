## trauso

> cargo build --release

# AGENTS.md - Trauso Development Guide

## Build Commands

### Tauri (React/Rust)
```bash
# Install dependencies (npm)
npm install

# Development mode (hot reload)
npm run tauri dev

# Build for production (output: src-tauri/target/release/)
npm run tauri build

# Preview production build
npm run preview

# Run Vite dev server only (for frontend testing)
npm run dev

# Build Vite bundle only
npm run build
```

### Rust Backend
```bash
cd src-tauri/

# Run tests
cargo test

# Check code (faster than build)
cargo check

# Build release
cargo build --release

# Update dependencies
cargo update
```

## Testing

**Current Status**: No formal test framework configured.

### Recommendations:
- **Frontend (TypeScript/React)**: Consider adding `vitest` or `@testing-library/react`
- **Backend (Rust)**: Use `cargo test` (already available)
- **E2E**: Consider `tauri-driver` with Playwright or Cypress

## Code Style Guidelines

### TypeScript/React (Frontend)

#### Import Style
- Group imports: React → Third-party → Local components
- Named imports preferred for better tree-shaking
- Use `@/` alias for absolute imports from `src/` directory

```typescript
import { useState, useEffect, useCallback } from 'react'
import { Button } from '@/components/ui/button'
import { DownloadProgress } from './DownloadProgress'
```

#### Component Style
- Use functional components with hooks
- Props should be typed with interfaces
- Use TypeScript strict mode
- Use `export` explicitly for components meant to be used outside module

```typescript
interface DownloadProgressProps {
  progress: number
  status: 'downloading' | 'paused' | 'completed' | 'error'
  onPause: () => void
  onResume: () => void
  onCancel: () => void
}

export function DownloadProgress({
  progress,
  status,
  onPause,
  onResume,
  onCancel
}: DownloadProgressProps) {
  // Component implementation
}
```

#### Hooks Pattern
- Extract complex logic into custom hooks
- Use `useCallback` for event handlers passed to children
- Use `useMemo` for expensive computations
- Use `useEffect` for side effects (API calls, subscriptions)

```typescript
// Custom hook example
export function useDownloadQueue() {
  const [queue, setQueue] = useState<DownloadItem[]>([])
  const [status, setStatus] = useState<QueueStatus>('idle')

  const addToQueue = useCallback((item: DownloadItem) => {
    setQueue(prev => [...prev, item])
  }, [])

  return { queue, status, addToQueue }
}
```

#### Error Handling
- Use Error Boundaries for React component errors
- Use try-catch for async operations
- Display user-friendly error messages
- Log errors to console and/or error tracking service

```typescript
try {
  const result = await api.getTeraboxInfo(url)
  setResult(result)
} catch (error) {
  console.error('Failed to fetch TeraBox info:', error)
  setError('Gagal mengambil informasi file. Coba lagi.')
}
```

#### State Management
- Prefer local component state with `useState`
- For complex state, consider `useReducer` or Zustand
- Keep state as close to where it's used as possible

#### CSS/Tailwind
- Use Tailwind CSS utility classes
- Follow component-based structure
- Use `cn()` helper from `clsx` and `tailwind-merge` for conditional classes

```typescript
import { cn } from '@/lib/utils'

export function Button({ variant = 'default', className, ...props }) {
  return (
    <button
      className={cn(
        'px-4 py-2 rounded font-medium',
        variant === 'default' && 'bg-blue-600 text-white',
        variant === 'ghost' && 'hover:bg-gray-100',
        className
      )}
      {...props}
    />
  )
}
```

### Rust (Tauri Backend)

#### Naming Conventions
- Functions/Variables: `snake_case`
- Types/Structs: `PascalCase`
- Constants: `UPPER_CASE`
- Modules: `snake_case`

```rust
// Function
pub fn get_terabox_info(url: &str) -> Result<TeraboxInfo, Error> {
    // Implementation
}

// Struct
pub struct TeraboxInfo {
    pub filename: String,
    pub size: u64,
    pub url: String,
}

// Constant
const MAX_RETRIES: u32 = 3;
```

#### Error Handling
- Use `Result<T, E>` for fallible operations
- Use `?` operator for early returns
- Avoid `unwrap()` in production code (use `expect()` with clear message if needed)
- Create custom error types with `thiserror` crate

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum TeraboxError {
    #[error("Failed to fetch URL: {0}")]
    FetchError(#[from] reqwest::Error),

    #[error("Invalid response format")]
    InvalidResponse,

    #[error("File not found")]
    NotFound,
}

pub async fn get_download_link(url: &str) -> Result<String, TeraboxError> {
    let response = reqwest::get(url).await?;
    // Process response
    Ok(download_url)
}
```

#### Async/Await
- Use `tokio` for async runtime
- Use `async fn` for async functions
- Use `.await` properly (avoid blocking operations)

```rust
use tauri::command;

#[command]
pub async fn start_download(url: String) -> Result<String, String> {
    // Async operation
    let result = fetch_url(&url).await
        .map_err(|e| e.to_string())?;
    Ok(result)
}
```

#### Tauri Commands
- Use `#[command]` attribute for exposed functions
- Return `Result<T, String>` for error handling (Tauri serializes as error)
- Use appropriate types for parameters (String, numbers, structs with Serialize)

```rust
#[tauri::command]
pub async fn add_download(
    url: String,
    dir: Option<String>,
    filename: Option<String>,
) -> Result<String, String> {
    let download_dir = dir.unwrap_or_else(|| get_default_dir());
    // Implementation
    Ok(gid)
}
```

## Project Architecture

### Technology Stack
- **Frontend**: React 18 + TypeScript + Vite
- **Backend**: Rust (Tauri v2)
- **Styling**: Tailwind CSS v3
- **UI Components**: Radix UI + Shadcn/ui
- **Download Engine**: aria2 (multi-connection RPC client)
- **Build Tool**: Vite (frontend) + Cargo (backend)

### Directory Structure
```
trauso/
├── src/                          # React frontend
│   ├── main.tsx                 # React entry point
│   ├── App.tsx                  # Main App component
│   ├── components/              # Reusable UI components
│   │   ├── file-list.tsx       # File browser with tree view
│   │   ├── download-progress.tsx # Download progress with controls
│   │   ├── settings-dialog.tsx  # Settings panel
│   │   └── ui/                  # Shadcn UI components
│   ├── lib/                     # Utilities and helpers
│   │   ├── api.ts               # Tauri command wrapper
│   │   ├── types.ts             # TypeScript type definitions
│   │   └── utils.ts             # Helper functions
│   ├── hooks/                   # Custom React hooks
│   ├── assets/                  # Static assets (images, fonts)
│   └── index.css                # Global styles + Tailwind directives
├── src-tauri/                   # Rust backend
│   ├── src/
│   │   ├── lib.rs               # Main entry point
│   │   ├── main.rs              # Tauri app entry
│   │   ├── terabox/             # TeraBox API module
│   │   │   ├── mod.rs
│   │   │   └── api.rs          # get_info, get_download_link
│   │   ├── aria2/               # aria2 RPC client
│   │   │   ├── mod.rs
│   │   │   └── client.rs       # add_download, get_status, pause, resume, cancel
│   │   └── download/            # Download management module
│   ├── Cargo.toml               # Rust dependencies
│   ├── tauri.conf.json          # Tauri configuration
│   └── target/                  # Build output (release binary)
├── aria2/                       # aria2 binary (Windows)
│   └── aria2c.exe
├── aria2.conf                   # aria2 configuration
├── public/                      # Static assets served by Vite
├── package.json                 # Node.js dependencies
├── tsconfig.json                # TypeScript configuration
├── vite.config.ts               # Vite bundler configuration
├── tailwind.config.js           # Tailwind CSS configuration
└── postcss.config.js            # PostCSS configuration
```

### Tauri Commands (Backend → Frontend)
```typescript
// TeraBox API
get_terabox_info(url: string)              → Promise<TeraboxInfo>
get_download_link(params: DownloadParams)  → Promise<DownloadLink>
extract_shorturl(url: string)              → Promise<string>

// aria2 Download Management
start_aria2()                               → Promise<void>
add_download(url: string, dir?: string, filename?: string)    → Promise<string>
get_download_status(gid: string)            → Promise<DownloadStatus>
pause_download(gid: string)                 → Promise<void>
resume_download(gid: string)                → Promise<void>
cancel_download(gid: string)                → Promise<void>
get_all_downloads()                         → Promise<DownloadStatus[]>

// Settings & Config
set_download_dir(path: string)              → Promise<void>
get_download_dir()                          → Promise<string>
set_download_server(server: 1 | 2)          → Promise<void>
get_download_server()                       → Promise<number>
set_download_mode(mode: 'sequential' | 'parallel') → Promise<void>
get_download_mode()                         → Promise<string>
```

### Data Flow
```
User Action (UI)
    ↓
Frontend (React Components)
    ↓
API Layer (src/lib/api.ts)
    ↓
Tauri Commands (invoke())
    ↓
Rust Backend (src-tauri/src/)
    ↓
External Services (TeraBox API, aria2 RPC)
```

## Development Workflow

### 1. Setup (First Time)
```bash
# Install Node.js dependencies
npm install

# Install Rust (if not already installed)
# Visit: https://www.rust-lang.org/tools/install

# Verify Tauri CLI
npm run tauri -- info
```

### 2. Development
```bash
# Start development server (hot reload for both frontend and backend)
npm run tauri dev

# Frontend changes only (faster reload)
npm run dev

# Backend changes only
cd src-tauri && cargo watch -x run
```

### 3. Building
```bash
# Build for production
npm run tauri build

# Output location:
# - Windows: src-tauri/target/release/trauso.exe
# - macOS: src-tauri/target/release/Trauso.app
# - Linux: src-tauri/target/release/trauso
```

### 4. Testing Before Commit
- **Frontend**: Test UI changes in dev mode, check TypeScript errors
- **Backend**: Run `cargo test` and `cargo clippy`
- **Full Build**: Run `npm run tauri build` to verify release build works

### 5. Common Tasks
- **Add new Tauri command**: Define function in `src-tauri/src/lib.rs` with `#[command]`
- **Add new UI component**: Create in `src/components/`, export and import in `App.tsx`
- **Update types**: Modify `src/lib/types.ts` for type consistency across frontend
- **Styling**: Use Tailwind utility classes, extend theme in `tailwind.config.js`

## Notes

### Dependencies
- **Node.js** v18+ required for frontend development
- **Rust** (latest stable) required for backend development
- **C++ Build Tools** required on Windows for Tauri build
- **aria2 binary** must be present in `aria2/` folder (bundled with release)

### Configuration
- **Tauri config**: `src-tauri/tauri.conf.json` (app name, version, permissions)
- **Tailwind config**: `tailwind.config.js` (theme, plugins, content paths)
- **TypeScript config**: `tsconfig.json` (compiler options, paths)
- **aria2 config**: `aria2.conf` (download settings, connections)

### Current Limitations
- No formal testing framework configured (consider adding vitest/jest)
- No linting/formatting configured (consider ESLint + Prettier)
- Error tracking not integrated (consider Sentry)

### Best Practices
- Keep components small and focused (single responsibility)
- Use TypeScript strict mode for type safety
- Handle errors gracefully in both frontend and backend
- Use Tauri's native dialogs for file operations
- Optimize bundle size by avoiding unnecessary dependencies
- Test aria2 RPC calls before implementing complex download logic

---
> Source: [basstimam/trauso](https://github.com/basstimam/trauso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
