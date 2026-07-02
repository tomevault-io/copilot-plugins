## hand-write

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a pnpm monorepo containing multiple packages:

- **lc/**: LeetCode problem solutions (TypeScript/JavaScript)
- **react-hand-write/**: Custom React hooks library with WebWorker utilities
- **micro-fronted/**: Minimal micro-frontend framework
- **playground/**: React + TypeScript + Vite demo application
- **vue-playground/base/**: Vue 3 + TypeScript + Vite demo application

## Development Commands

### Root Level
```bash
# Start React playground (main demo environment)
pnpm dev
# or explicitly:
pnpm dev:playground

# Build React playground
pnpm build:playground

# Start backend server for video upload
pnpm server
```

### React Playground (playground/)
```bash
# Development server (port typically 5173)
pnpm dev

# Production build with TypeScript checking
pnpm build

# Lint code
pnpm lint

# Preview production build
pnpm preview

# Start backend upload server (Express on port 3001)
pnpm server

# Start backend server in watch mode
pnpm server:watch
```

### Vue Playground (vue-playground/base/)
```bash
# Development server
pnpm dev

# Production build with type checking (parallel execution)
pnpm build

# Type checking only (no build)
pnpm type-check

# Lint and auto-fix
pnpm lint

# Format code with Prettier
pnpm format

# Preview production build
pnpm preview
```

## Key Architecture Patterns

### React Custom Hooks Library (react-hand-write/)

#### Core Hooks
- **useLastest**: Returns a ref that always contains the latest value, solving closure staleness issues
- **useUpdate**: Forces component re-renders via setState
- **useRefValue**: Ref-based value management

#### Advanced WebWorker-Based Hooks

**useVideoAnalysis** - Video frame analysis using WebWorker for non-blocking UI:
- Analyzes video frames for brightness, color distribution, scene changes
- Generates thumbnails at key frames
- Returns: `{ isAnalyzing, progress, currentAnalysis, analysisResult, thumbnails, startAnalysis, stopAnalysis, reset }`
- Worker communication pattern: posts `ANALYZE_FRAME`, `GENERATE_THUMBNAIL`, `EXTRACT_KEYFRAMES` messages
- Expected worker at `/workers/videoAnalysisWorker.js`

**useSplitVideo** - Large video chunked upload with MD5 calculation:
- Uses `MD5WorkerPool` for parallel MD5 computation across multiple workers
- Chunks files into configurable sizes (default 10MB)
- Manages upload state: `idle`, `preparing`, `calculating-md5`, `uploading`, `merging`, `completed`, `error`, `paused`, `cancelled`
- Returns: `{ uploadFile, chunkFile, md5Progress, isPaused, error, isUploading, status, progress }`

**useVideoLoader** - Video loading with progress tracking:
- Supports both File objects and URLs
- Tracks loading progress and metadata extraction
- Returns: `{ isLoading, isLoaded, progress, error, metadata, videoElement, loadVideo, cancelLoad, retry, reset }`

#### Worker Pool Pattern (worker/workpool.ts)

The repository implements a reusable Worker Pool pattern for parallel computation:
- `WorkerPool`: Generic worker pool with task queue and worker reuse
- `MD5WorkerPool`: Specialized pool for MD5 calculation tasks using SHA-256
- Workers are initialized once and reused to avoid creation/destruction overhead
- Task queuing system with progress callbacks
- Worker lifecycle: `initialize()` → `execute()` → `releaseWorker()` → `destroy()`
- Worker code is embedded as string constant (`MD5_WORKER_CODE`), converted to Blob URL for worker instantiation
- Supports concurrent task execution with configurable max workers (default: 2)
- Progress tracking via `onProgress` callbacks during file reading and hashing

### Micro-Frontend Framework (micro-fronted/)

Simple micro-frontend implementation:
- `registerMicroApps(apps)`: Registers micro-apps array
- `start()`: Initializes router matching via `handleRouter()`
- `getApps()`: Retrieves registered apps
- Core files: `index.ts` (app registry), `handleRoute.ts` (routing logic)

### LeetCode Solutions (lc/)

Algorithm implementations covering:
- **Arrays**: twoSum, threeSum, moveZeros, maxSubArray, rotate, merge
- **Binary Search**: search, searchRange, findMin, findMin_duplicate, searchInsert, searchMatrix
- **Linked Lists**: reverseList, getIntersectionNode, mergeTwoLists, addTwoNumbers, removeNthFromEnd, swapPairs, hasCycle, sortList
- **Trees**: toTree, inorderTraversal, invertTree, diameterOfBinaryTree
- **Hash Tables**: groupAnagrams, singleNumber-hashmap, longestConsecutive
- **Other**: isPalindrome, maxArea, produceExpectSelf, subArraySum, findMedianSortedArray

## Backend Server Architecture

### Video Upload Server (playground/node/)

Express-based backend for chunked video file uploads with the following REST API:

**Server Entry Point**: `server.ts` (Port 3001)
- CORS enabled for cross-origin requests
- Static file serving at `/uploads`
- All upload routes prefixed with `/api/upload`

**API Endpoints** (defined in `video.ts`):
1. **POST `/api/upload/check`** - Fast upload check (MD5-based file existence verification)
   - Request: `{ fileMD5: string, filename: string }`
   - Response: `{ exists: boolean, path?: string, size?: number, md5?: string }`

2. **POST `/api/upload/chunk`** - Upload single chunk
   - Form data with file blob, `fileId`, `chunkIndex` (passed as query params for multer compatibility)
   - Chunks stored in `uploads/chunks/{fileId}/chunk_{index}`

3. **GET `/api/upload/chunks/:fileId`** - Query uploaded chunks (for resumable uploads)
   - Response: `{ chunks: number[], total: number }`

4. **POST `/api/upload/merge`** - Merge all chunks into final file
   - Request: `{ fileId: string, totalChunks: number, filename: string }`
   - Validates chunk completeness, merges sequentially, calculates MD5, cleans up chunks
   - Response: `{ path: string, filename: string, size: number, md5: string }`

5. **DELETE `/api/upload/:fileId`** - Clean up incomplete upload

**Storage Structure**:
- `uploads/chunks/{fileId}/` - Temporary chunk storage
- `uploads/videos/` - Final merged video files
- Files named with timestamp to avoid collisions: `{basename}_{timestamp}{ext}`

**Key Implementation Details**:
- Uses `multer` for multipart form handling with custom diskStorage configuration
- MD5 calculation uses Node.js crypto module with streaming for memory efficiency
- Chunks validated before merge to ensure completeness

## Important Notes

### Package Manager
- Uses **pnpm workspaces** exclusively (see `pnpm-workspace.yaml`)
- Workspace packages: `playground`, `vue-playground/base`, `react-hand-write`
- Use `pnpm --filter <package>` to run commands in specific packages

### WebWorker Integration
- Worker code embedded as string constants in `workpool.ts` (not separate files in `public/`)
- Worker Pool pattern avoids repeated worker creation for performance
- Progress callbacks allow UI updates during long-running worker tasks
- Uses browser's native `crypto.subtle.digest()` for SHA-256 hashing (labeled as MD5 in variable names)

### Video Upload Flow (Frontend ↔ Backend)
1. **File Selection** → Calculate MD5 using `MD5WorkerPool` (client-side, parallel workers)
2. **Fast Upload Check** → POST `/api/upload/check` to see if file already exists
3. **Chunking** → Split file into configurable chunks (default 10MB)
4. **Resumable Upload** → GET `/api/upload/chunks/:fileId` to check existing chunks, then upload missing chunks with concurrency control
5. **Merge** → POST `/api/upload/merge` after all chunks uploaded
6. **Status Management** → Track upload states: `idle` → `calculating-md5` → `checking` → `preparing` → `uploading` → `merging` → `completed`

### Video Processing Demo (playground/src/VideoLoaderDemo.tsx)
- Three-tab interface: "视频加载" (load), "视频分析" (analysis), "大视频分片" (split)
- Demonstrates integration of all three video-related hooks
- Auto-switches to analysis tab when video loads
- Shows real-time progress, frame analysis, thumbnails, and upload chunks

---
> Source: [CathLee/hand-write](https://github.com/CathLee/hand-write) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
