## arrgh-excalidraw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development server with Turbopack
npm run dev

# Production build
npm run build

# Start production server
npm start

# Lint code
npm run lint

# Run tests
npm run test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage report
npm run test:coverage
```

## Architecture Overview

This is a **Next.js 15 application** that integrates **AI-powered diagram generation with Excalidraw**. The system has two main components:

### Frontend (Next.js)
- **Main Component**: `src/components/excalidraw-with-ai.tsx` - Excalidraw canvas with AI generation modal
- **API Route**: `src/app/api/generate/route.ts` - Proxy to FastAPI backend
- **UI Components**: shadcn/ui components in `src/components/ui/`

### Backend Integration
- **FastAPI Service**: Separate service with `/api/generate-excalidraw` endpoint
- **Reference Implementation**: `fastapi-endpoint-simplified.py` shows the required FastAPI endpoint structure

## Key Technical Details

### Excalidraw Integration
- Excalidraw is **dynamically imported** (`ssr: false`) to prevent SSR issues
- Components must handle the `mounted` state before rendering Excalidraw
- API responses must contain `elements` array and `appState` object in Excalidraw JSON format

### Environment Variables
- `FASTAPI_URL`: Backend service URL (defaults to `http://localhost:8000`)
- `FASTAPI_TOKEN`: Optional bearer token for backend authentication

**Production Notes:**
- Custom domain (https://excalidraw.paulbonneville.com) is handled via DNS configuration
- No additional environment variables required for custom domain setup

### AI Generation Flow
1. User enters prompt in modal dialog
2. Frontend sends POST to `/api/generate`
3. Next.js API route proxies to FastAPI backend
4. FastAPI returns Excalidraw JSON elements
5. Frontend updates Excalidraw scene with generated elements

## Deployment Architecture
- **Frontend**: Next.js app on Google Cloud Run
- **Backend**: Existing arrgh-fastapi service with new diagram endpoint
- **Production URL**: https://excalidraw.paulbonneville.com (custom domain)
- **Deployment**: Both services deployed independently, connected via `FASTAPI_URL`
- **Health Checks**: Deployment workflow tests both Cloud Run URL and custom domain
- **DNS Configuration**: Custom domain configured via DNS records (not Cloud Run domain mapping)

## Technology Stack
- Next.js 15 with React 19 and TypeScript
- Excalidraw for diagram editing
- shadcn/ui + Tailwind CSS for UI
- Lucide React for icons

## GitHub CLI Permissions

This project requires specific GitHub CLI permissions for Claude Code to manage issues, projects, and CI/CD workflows.

### Required Scopes
```bash
# Essential scopes for full project management
repo                    # Full repository access (read/write code, issues, PRs)
project                 # GitHub Projects v2 access (create/edit project items)
workflow                # GitHub Actions access (view/trigger workflows)
read:org                # Organization access (for org-level projects)
```

### Quick Permission Restoration
If Claude Code loses permissions, run these commands to restore access:

```bash
# Standard refresh with all required scopes
gh auth refresh -s repo -s project -s workflow -s read:org

# Alternative: Interactive refresh (will prompt for scopes)
gh auth refresh

# Verify current permissions
gh auth status
```

### Troubleshooting GitHub CLI Issues

**Permission Denied Errors:**
```bash
# Check current scopes
gh auth status

# Refresh with missing scopes
gh auth refresh -s [missing-scope]
```

**Project Access Issues:**
```bash
# Verify project exists and is accessible
gh project list --owner [username]

# View project details
gh project view [number] --owner [username]
```

**Common Commands for Project Management:**
```bash
# Create issue and add to project
gh issue create --title "Issue Title" --body "Description" --project "Project Name"

# Add existing issue to project
gh project item-add [project-number] --owner [username] --url [issue-url]

# List project items
gh project item-list [project-number] --owner [username]
```

### Personal Access Token Alternative

For more stable, long-term access, consider using a Personal Access Token (PAT):

1. **Create PAT**: GitHub Settings → Developer settings → Personal access tokens
2. **Required Scopes**: `repo`, `project`, `workflow`, `read:org`
3. **Configure CLI**: `gh auth login --with-token < token.txt`
4. **Expiration**: Set to 90 days or 1 year for stability

**PAT Management:**
- Store token securely
- Set calendar reminder before expiration
- Keep backup refresh commands available

### Quick Reference
```bash
# Emergency permission restore
gh auth refresh -s repo -s project -s workflow -s read:org

# For new token setup or renewal
gh auth login --with-token

# Check everything is working (replace 'pbonneville' with your username)
gh auth status && gh project list --owner pbonneville

# Test project access (replace 'pbonneville' with your username)
gh project view 1 --owner pbonneville
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Paul-Bonneville-Labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
