## project-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Project Control은 프로젝트/태스크 관리 웹앱. React + Vite + Supabase 기반. Korean UI. Vercel 배포 (커밋+푸시 → 자동 배포).

## Commands

- `npm run dev` — Vite dev server at http://localhost:5173
- `npm run build` — Build React app to `dist/`

No test framework configured.

## Tech Stack

- **React 18** (JavaScript/JSX, no TypeScript)
- **Vite 5** build tool
- **Supabase** for auth, database, user management
- **lucide-react** for icons
- **Pretendard** font (Korean-optimized, CDN)
- No routing library, no state management library, no UI component library

## Architecture

### State Management
All state in `src/App.jsx` via `useState`/`useCallback`. Prop-drilled. No real-time subscriptions — refetch after mutation.

### View Switching
`activeView` state: dashboard, project, archive. No router.

### Component Structure
```
App.jsx              — Central state, all CRUD handlers
├── Auth.jsx         — Login/signup with invite code
├── Sidebar.jsx      — "프로젝트" 클릭→대시보드, 프로젝트 리스트, 보관함
├── Dashboard.jsx    — 통계 카드 + 프로젝트별 칸반 메이슨리 보드
├── ProjectDetail.jsx — 탭: 대시보드(태스크/인포) | 업무추진표(RoadmapView)
│   └── RoadmapView.jsx — 분기별 업무추진표 (TASK 아코디언 + 테이블)
├── ArchiveView.jsx  — 보관된 프로젝트/태스크 복원/삭제
├── SettingsPanel.jsx — 설정, 관리자 패널 (승인/초대코드/관리자 승격)
└── Modals: ProjectModal, CategoryModal, TaskModal, TaskViewModal, InfoModal
```

### Supabase Data Model
- **projects** — name, description, color, category_id, archived, sort_order
- **project_categories** — name, sort_order
- **tasks** — project_id, title, assignee, priority, archived, started_at
- **task_items** — task_id, content, completed, completed_at
- **project_infos** — project_id, type, label, value
- **user_profiles** — email, name, role (user/admin), approved, invite_code
- **invite_codes** — code, created_by, active
- **roadmap_rows** — project_id, major, minor, assignee, output, sort_order
- **roadmap_cells** — row_id, year, month, content (JSON checklist)
- **style_guide** — content

All tables use `user_id` + RLS (`WITH CHECK`) for row-level isolation.

### Auth Flow
Supabase email/password. Invite code → auto-approve. Otherwise admin approval. Admin can promote/demote users.

## Styling

Custom CSS only. Design tokens as CSS variables in `src/style.css`. See `STYLE_GUIDE.md`.

Key layout: min-width 1200px, sidebar 310px fixed, content flex: 1, content-header padding 70px top.

### CSS Variable Notes
- Font sizes: `--font-12` (12px), `--font-13`, `--font-14`, etc. (`--font-xs`, `--font-sm` 미정의 — 사용금지)
- Font weights: `--weight-500`, `--weight-600`, `--weight-700` (`--weight-medium`, `--weight-semibold` 미정의 — 사용금지)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/reverve9)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/reverve9)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
