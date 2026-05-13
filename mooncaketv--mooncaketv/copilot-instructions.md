## mooncaketv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

MoonCakeTV (月饼TV) is a **simplified** Next.js 15 web application for video streaming aggregation and search. This is one of several independent repositories in the MoonCake TV multi-repository workspace.

**Architecture Philosophy:**

- **Single user mode** - No multi-user complexity
- **Optional password** - Public access or password-protected
- **File-based storage** - No database required
- **VPS-first** - Simple deployment without Docker
- **Minimal dependencies** - Only what's necessary

**Tech Stack:**

- Next.js 15 with App Router and React 19
- TypeScript 5.x
- Tailwind CSS 4.x
- **File-based JSON storage** (no database)
- JWT authentication with `jose`
- bcryptjs for password hashing
- Radix UI components with shadcn/ui
- Vidstack/HLS.js for video playback
- Zustand for state management
- Zod v4 for validation

**Node.js Requirement:** >=22.0.0

## TypeScript Configuration

**Path Aliases:**

- `@/*` maps to `./src/*` (e.g., `@/components`, `@/lib`)
- `~/*` maps to `./public/*` (e.g., `~/logo.png`)

Always use these aliases instead of relative imports for better maintainability.

## Common Development Commands

```bash
# Development
npm install              # Install dependencies
npm run dev              # Start dev server on 0.0.0.0:3333
npm run build            # Production build
npm run start            # Start production server

# Code Quality
npm run lint             # Run ESLint
npm run lint:fix         # Fix ESLint errors and format code
npm run typecheck        # TypeScript type checking
npm run format           # Format code with Prettier
npm run format:check     # Check formatting without changes

# Testing
npm run test             # Run Jest tests
npm run test:watch       # Run tests in watch mode

# Git Deployment
make origin              # Push to origin remote
make tea                 # Push to tea remote
make vercel              # Push to vercel remote

# Simple Commands
make dev                 # npm run dev
make build               # npm run build
```

## Authentication Architecture

The application uses a **simplified single-user authentication** system with optional password protection.

### How It Works

**Two modes:**

1. **No password set** → Anyone can access (public mode)
2. **Password set** → Login required for all protected routes

**Middleware flow (`src/middleware.ts`):**

1. Check if password has been set (via `isPasswordSet()`)
2. If no password → Allow access (skip authentication)
3. If password exists → Require `mc-auth-token` JWT cookie
4. Redirect to `/login` if not authenticated

**Protected vs Public paths:**

- Public: `/_next`, `/favicon.ico`, `/login`, `/api/login`, `/api/logout`, `/api/server-config`, static files
- Protected: All other routes (when password is set)

### Setting a Password

Visit `/login` page:

- If no password is set, you can set one
- Password is hashed with bcryptjs and stored in `data/user-data.json`
- Once set, the app requires login for all protected routes

### Removing Password Protection

Delete or edit `data/user-data.json` and remove the `password_hash` value.

**Warning:** This will also delete bookmarks and watch history unless you backup the file first.

## Data Storage

All user data is stored in a single JSON file: `data/user-data.json`

**File structure:**

```json
{
  "password_hash": "",
  "bookmarks": [
    {
      "id": "video_123",
      "title": "电影名称",
      "thumbnail": "https://...",
      "url": "https://...",
      "added_at": "2025-01-15T10:30:00.000Z"
    }
  ],
  "watch_history": [
    {
      "id": "video_123",
      "title": "电影名称",
      "progress": 120,
      "last_watched": "2025-01-15T10:30:00.000Z"
    }
  ],
  "settings": {}
}
```

### Data Management

**Backup:**

```bash
cp data/user-data.json backup/user-data-$(date +%Y%m%d).json
```

**Restore:**

```bash
cp backup/user-data-20250115.json data/user-data.json
```

**Migration to new server:**

```bash
# Old server
scp data/user-data.json user@new-server:/path/to/mooncaketv-web/data/

# New server - data is ready!
```

**Reset everything:**

```bash
rm data/user-data.json
# File will be recreated with defaults on next startup
```

## Project Structure

### Key Directories

- **`data/`** - Data storage directory
  - `user-data.json` - All user data (auto-created)

- **`src/app/`** - Next.js App Router pages and layouts
  - `api/` - API route handlers
    - `login/` - User login endpoint
    - `logout/` - User logout endpoint
    - `bookmarks/` - Bookmarks CRUD API
    - `history/` - Watch history API
    - `server-config/` - Server configuration endpoint
    - `image-proxy/` - Image proxy for CORS handling
    - `speed-test/` - Network speed testing endpoint
  - `play/` - Video playback page
  - `search/` - Search results page
  - `bookmarks/` - User bookmarks page
  - `watch-history/` - Watch history page
  - `settings/` - User settings page
  - `login/` - Login page

- **`src/components/`** - React components
  - `ui/` - shadcn/ui components (Radix UI primitives)
  - `common/` - Shared common components
  - `mc-play/` - Video player components (Vidstack-based)
  - `sidebar/` - Sidebar navigation components
  - `home/` - Homepage-specific components
  - `search-page/` - Search page components
  - `mc-search/` - Search functionality components
  - `mobile/` - Mobile-specific UI components
  - `douban/` - Douban integration components
  - `logo/` - Logo components

- **`src/lib/`** - Library utilities
  - `file-storage.ts` - **Core data persistence layer** (bookmarks, history, settings)
  - `simple-auth.ts` - **Authentication utilities** (password verification, JWT)
  - `utils.ts` - General utility functions (including cn for className merging)
  - `admin.types.ts` - Admin-related type definitions

- **`src/stores/`** - Zustand state stores
  - `global.ts` - Global application state
  - `user.ts` - User state management
  - `sidebar.ts` - Sidebar state

- **`src/hooks/`** - Custom React hooks

- **`src/types/`** - TypeScript type definitions

## Core Architecture

### File Storage System (`src/lib/file-storage.ts`)

The file storage system provides persistent storage for user data using JSON files.

**Key functions:**

```typescript
// Read all user data
export async function readUserData(): Promise<UserData>;

// Write user data (atomic operation)
export async function writeUserData(data: UserData): Promise<void>;

// Bookmarks
export async function addBookmark(bookmark: Bookmark): Promise<void>;
export async function removeBookmark(id: string): Promise<void>;
export async function getBookmarks(): Promise<Bookmark[]>;

// Watch History
export async function addWatchHistory(item: WatchHistoryItem): Promise<void>;
export async function updateWatchProgress(
  id: string,
  progress: number,
): Promise<void>;
export async function getWatchHistory(): Promise<WatchHistoryItem[]>;
export async function clearWatchHistory(): Promise<void>;
```

**Atomic writes:**
All write operations use atomic file writes (write to temp file, then rename) to prevent data corruption.

### Authentication System (`src/lib/simple-auth.ts`)

Simple password-based authentication with JWT tokens.

**Key functions:**

```typescript
// Check if password has been set
export async function isPasswordSet(): Promise<boolean>;

// Set new password (hashed with bcryptjs)
export async function setPassword(password: string): Promise<void>;

// Verify password against stored hash
export async function verifyPassword(password: string): Promise<boolean>;

// Generate JWT token
export async function generateAuthToken(): Promise<string>;

// Verify JWT token
export async function verifyAuthToken(token: string): Promise<boolean>;
```

## API Routes

### Authentication

**POST /api/login**

```typescript
Request: { password: string }
Response: { code: 200, message: "登录成功" }
Sets cookie: mc-auth-token (JWT, 7 days)
```

**POST /api/logout**

```typescript
Response: { code: 200, message: "退出登录成功" }
Clears cookie: mc-auth-token
```

### Bookmarks

**GET /api/bookmarks**

```typescript
Response: {
  code: 200,
  data: Bookmark[]
}
```

**POST /api/bookmarks**

```typescript
Request: {
  id: string,
  title: string,
  thumbnail: string,
  url: string
}
Response: { code: 200, message: "添加收藏成功" }
```

**DELETE /api/bookmarks?id={id}**

```typescript
Response: { code: 200, message: "删除收藏成功" }
```

**PATCH /api/bookmarks**

```typescript
Request: { id: string, updates: Partial<Bookmark> }
Response: { code: 200, message: "更新收藏成功" }
```

### Watch History

**GET /api/history**

```typescript
Response: {
  code: 200,
  data: WatchHistoryItem[]
}
```

**POST /api/history**

```typescript
Request: {
  id: string,
  title: string,
  progress: number
}
Response: { code: 200, message: "添加观看记录成功" }
```

**DELETE /api/history**

```typescript
Response: { code: 200, message: "清空观看历史成功" }
```

**PATCH /api/history**

```typescript
Request: {
  id: string,
  progress: number
}
Response: { code: 200, message: "更新观看进度成功" }
```

## Common Patterns

### Adding a New API Route

Create a route handler in `src/app/api/{route-name}/route.ts`:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { HTTP_STATUS } from "@/config/constants";

export async function GET(request: NextRequest) {
  try {
    // Handle GET request
    return NextResponse.json({
      code: HTTP_STATUS.OK,
      data: {
        /* ... */
      },
    });
  } catch (error) {
    return NextResponse.json({
      code: HTTP_STATUS.INTERNAL_SERVER_ERROR,
      message: "服务器错误",
    });
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    // Handle POST request
    return NextResponse.json({
      code: HTTP_STATUS.OK,
      message: "操作成功",
    });
  } catch (error) {
    return NextResponse.json({
      code: HTTP_STATUS.BAD_REQUEST,
      message: "请求参数错误",
    });
  }
}
```

### Working with File Storage

```typescript
import { addBookmark, getBookmarks } from "@/lib/file-storage";

// Add a bookmark
await addBookmark({
  id: "video_123",
  title: "电影名称",
  thumbnail: "https://example.com/thumb.jpg",
  url: "https://example.com/video",
});

// Get all bookmarks
const bookmarks = await getBookmarks();
```

### Schema Validation

Use Zod v4 with validation error formatting:

```typescript
import { z } from "zod";
import { fromZodError } from "zod-validation-error";

const schema = z.object({
  id: z.string().min(1),
  title: z.string().min(1),
});

try {
  const data = schema.parse(input);
} catch (error) {
  const validationError = fromZodError(error);
  console.error(validationError.toString());
}
```

### State Management with Zustand

```typescript
import { create } from "zustand";

interface MyStore {
  value: string;
  setValue: (value: string) => void;
}

export const useMyStore = create<MyStore>((set) => ({
  value: "",
  setValue: (value) => set({ value }),
}));
```

## Environment Variables

Only **one required** environment variable:

```bash
# JWT Secret (required)
JWT_SECRET=your_random_secret_key_here
```

Generate a random secret:

```bash
openssl rand -hex 32
```

**Optional variables:**

```bash
# Port (default: 3000)
PORT=3333

# Node environment
NODE_ENV=production
```

## Deployment

### VPS Deployment (Recommended)

**1. Clone and install:**

```bash
git clone https://github.com/your-repo/mooncaketv-web.git
cd mooncaketv-web
npm install
```

**2. Configure environment:**

```bash
cp .env.example .env
# Edit .env and set JWT_SECRET
```

**3. Build and run:**

```bash
npm run build
npm start
```

**4. Optional: Use PM2 for process management:**

```bash
npm install -g pm2
pm2 start npm --name "mooncaketv" -- start
pm2 startup
pm2 save
```

**5. Optional: Setup Nginx reverse proxy:**

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3333;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Vercel Deployment

```bash
# Push to vercel remote
make vercel

# Or use Vercel CLI
vercel --prod
```

Configure `JWT_SECRET` in Vercel environment variables.

## Security Notes

**Password Security:**

- Passwords are hashed with bcryptjs (10 rounds)
- Never stored in plain text
- JWT tokens expire after 7 days

**HTTPS Recommendation:**

- Use HTTPS in production (via Nginx + Let's Encrypt or Cloudflare)
- Protects JWT tokens and login credentials
- Required for secure cookie flags

**Data Privacy:**

- All data stored locally in `data/user-data.json`
- No external database or third-party storage
- Easy to backup and delete

**File Permissions:**
Ensure `data/` directory has appropriate permissions:

```bash
chmod 700 data/
chmod 600 data/user-data.json
```

## Video Player Components

The application uses Vidstack for video playback:

- **Vidstack** (`@vidstack/react`): Modern video player framework
- **HLS.js**: HLS streaming support in browsers
- **Video.js**: Alternative video player (legacy support)

Player components are in `src/components/mc-play/`.

## Development Tips

- Use `npm run lint:fix` to auto-fix linting issues and format code
- Type-check before committing: `npm run typecheck`
- For video playback testing, ensure CORS is enabled on source URLs
- The `data/` directory is git-ignored - your data stays private
- Backup `data/user-data.json` regularly if you have important bookmarks

## Troubleshooting

**Forgot password:**

```bash
# Option 1: Reset password (keeps bookmarks)
# Edit data/user-data.json and set password_hash to empty string ""
# Then visit /login to set new password

# Option 2: Delete everything (loses bookmarks)
rm data/user-data.json
```

**Corrupted data file:**

```bash
# Restore from backup
cp backup/user-data-20250115.json data/user-data.json

# Or start fresh
rm data/user-data.json
```

**Login issues:**

- Check that `JWT_SECRET` is set in `.env`
- Clear browser cookies and try again
- Check file permissions on `data/user-data.json`

## Migration from Old Version

If you're upgrading from the PostgreSQL/Redis version:

**See:** `SIMPLIFICATION_COMPLETE.md` for detailed migration guide

**Quick summary:**

1. Export bookmarks/history from old database
2. Convert to JSON format
3. Place in `data/user-data.json`
4. Set new password via `/login`

---
> Source: [MoonCakeTV/MoonCakeTV](https://github.com/MoonCakeTV/MoonCakeTV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
