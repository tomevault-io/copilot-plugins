## lifedesigncourse

> **When switching between projects, follow these steps:**

# lifedesigncourse.app Project Guidelines

## Project Switching Rule

**When switching between projects, follow these steps:**

1. Close all Chrome browser tabs and windows first
2. Open only the tabs relevant to the current project:
   - Vercel (deployment/hosting)
   - Firebase Console (backend/database)
   - Google Cloud Console (if applicable)
   - GitHub (repository)
   - Live site (the deployed application)

This ensures Chrome only shows tabs for the active project, avoiding confusion and maintaining a clean workspace.

---

## Project Overview

**lifedesigncourse.app** is a web application for delivering life design course content, enabling users to engage with course materials, track progress, and participate in community discussions.

---

## Key URLs & Resources

| Resource | URL |
|----------|-----|
| **Live Site** | https://lifedesigncourse.app |
| **GitHub Repository** | https://github.com/mdulin01/lifedesigncourse |
| **Firebase Console** | https://console.firebase.google.com/project/lifedesigncourse |
| **Vercel Dashboard** | https://vercel.com/dashboard |
| **Google Cloud Console** | https://console.cloud.google.com |

---

## Technical Stack

- **Frontend Framework:** React 18+
- **Build Tool:** Vite
- **Styling:** Tailwind CSS
- **Backend/Database:** Firebase (Firestore, Authentication, Storage)
- **Deployment:** Vercel
- **Version Control:** GitHub (mdulin01/lifedesigncourse)

---

## Project Structure

```
lifedesigncourse/
├── src/
│   ├── components/      # All React components (flat, no subdirectories)
│   ├── contexts/        # React contexts (BTSContext.jsx)
│   ├── hooks/           # Custom hooks (useMessages, useProgress, useTeams, etc.)
│   ├── constants.js     # App constants
│   ├── firebase-config.js  # Firebase initialization
│   ├── App.jsx          # Main app component
│   ├── main.jsx         # Entry point
│   └── index.css        # Global styles
├── public/              # Static assets
├── package.json         # Project dependencies
├── vite.config.js       # Vite configuration
├── tailwind.config.js   # Tailwind configuration
├── firestore.rules      # Firestore security rules
└── CLAUDE.md            # This file
```

---

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (typically http://localhost:5173)
npm run dev

# Build for production
npm run build

# Preview production build locally
npm run preview

# Lint code
npm run lint
```

---

## Infrastructure

- **Firebase Project ID:** lifedesigncourse
- **Firebase Storage Bucket:** `gs://lifedesigncourse.firebasestorage.app`
- **Database:** Firestore
- **Authentication:** Enabled (Google provider)
- **Storage:** Enabled for course materials and user uploads
- **Security Rules:** See `firestore.rules` in project root
- **Firebase config** is hardcoded in `src/firebase-config.js` (public API keys only)

## Architecture Notes

- **Key components:** Dashboard, CourseLanding, MyTeam, Journal, Workbook, CheckIn, ModuleProjects
- **Custom hooks:** useMessages, useProgress, useTeams, useUnread, useUserProfile, useWorkbook
- **Context:** BTSContext (behind-the-scenes content)
- All components are in a flat `src/components/` directory (no subdirectories)

---

## Deployment Notes

- **Hosting:** Vercel
- **Custom Domain:** lifedesigncourse.app
- **Branch Deployments:** Automatic from GitHub pushes
- **Production Build:** Auto-triggered on main/master branch commits

---

## File Scope Boundary

**CRITICAL: When working on this project, ONLY access files within the `lifedesigncourse/` directory.** Do not read, write, or reference files from any sibling project folder (dulinproperties, rainbow-rentals, downtownGSO, etc.). If you need something from another project, stop and ask first.

---
> Source: [mdulin01/lifedesigncourse](https://github.com/mdulin01/lifedesigncourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
