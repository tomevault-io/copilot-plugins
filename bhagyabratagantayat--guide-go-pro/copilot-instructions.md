## guide-go-pro

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Guide Go**

Guide Go is an Uber-like platform for booking local guides. It connects users with verified local guides through real-time geo-matching, interactive maps, and a seamless booking flow with role-based features.

**Core Value:** A friction-free, real-time booking experience that instantly connects tourists with nearby, verified local guides.

### Constraints

- **Tech stack**: Node/Express (backend), React/Tailwind (frontend), MongoDB Atlas (DB), Socket.io (Realtime). — Required by user for free scalable startup architecture.
- **Hosting**: Render (backend), Vercel (frontend), Cloudinary (media) — Required to keep free tier active.
- **Security**: JWT validation and robust role middlewares are required. Guide profiles must be administrator-verified before they can Go Live.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- JavaScript (Node.js backend, React frontend)
## Runtime & Frameworks
- **Backend:** Node.js, Express.js
- **Frontend:** React 19, Vite (Bundler)
## Key Dependencies
- **Database:** MongoDB via `mongoose`
- **State Management / Data Fetching:** `axios` (REST API), context API likely used locally (React)
- **Routing:** `react-router-dom`
- **Real-time:** `socket.io` (backend) & `socket.io-client` (frontend)
- **Authentication:** `jsonwebtoken` (JWT), `bcryptjs`
- **Mapping:** `leaflet`, `react-leaflet`
## Styling
- **CSS Framework:** TailwindCSS (`tailwindcss`, `postcss`, `autoprefixer`)
- **Icons:** `lucide-react`
## Configuration
- `package.json` at root, and in `backend/` and `frontend/`
- `.env` environment variable setup via `dotenv`
- `eslint` for linting
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Linting & Formatting
- **ESLint:** Frontend uses `eslint`, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh` to enforce React best practices. 
## Best Practices & Patterns
- **React Components:** Follow modern functional component patterns using React hooks (`useState`, `useEffect`, etc.).
- **Async/Await:** Used predominantly across both backend Node.js applications (for Mongoose and API calls) and frontend React applications (for `axios` fetch calls).
- **Styling:** Utility-first class structure driven by TailwindCSS instead of traditional CSS files, with standard class concatenation or standard template literals.
- **Environment config:** Secrets and environment specific configs reside in `.env` and are loaded via `dotenv` (backend) or `VITE_*` (frontend).
## Source Control
- Codebase utilizes Git. Branches/commits standard patterns should be applied.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## Client-Server Interaction
- **REST APIs:** Client makes HTTP requests to Node/Express backend (`/api/...`).
- **Real-time:** `socket.io` provides a bidirectional data flow for specific live features (likely used for live tracking, chat, or notifications).
## Frontend Architecture
- **Layering:** 
- **Entry point:** `main.jsx` and `App.jsx`.
## Backend Architecture
- **Layering (MVC pattern):**
- **Entry point:** `server.js`.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Source: [bhagyabratagantayat/guide-go-pro](https://github.com/bhagyabratagantayat/guide-go-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
