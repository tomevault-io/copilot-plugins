## ciso-work-cycle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow and Linear Integration (CRITICAL - READ FIRST)

### Linear Branch Naming Convention
**ALWAYS check Linear for expected branch name** - Linear auto-generates expected branch names in the `gitBranchName` field:
- **Never guess or assume branch names** - always check the issue details first using `mcp__Linear__get_issue`
- Example: `alex/for-129-create-visual-strategic-framework`
- **Format Pattern**: `alex/for-[issue-number]-[kebab-case-descriptive-title]`
- **Branch Lifecycle**: Once a PR is created, do NOT reuse that branch for new work - create new branches for new work

### Git Workflow (CRITICAL - ALWAYS FOLLOW)
- **NEVER push directly to main branch** - all changes must go through pull requests
- **ALWAYS create feature branches** for any changes, even simple fixes
- Use Linear branch names from `gitBranchName` field (check issue first with `mcp__Linear__get_issue`)
- Always commit and push changes to feature branches
- **ALWAYS create pull requests** for merging to main - no direct pushes allowed
- Use descriptive commit messages with Linear issue references and magic words
- **Line Number Preference**: Include specific line numbers in commit messages when referencing complex document changes for precise tracking
- Example: "Completes FOR-74: Add financial quantification and navigation to BISOPRO-02 Problem Statement"
- Example with line numbers: "Completes FOR-76: Transform BISOPRO-06 Authority Framework with executive summary (lines 3-26), navigation (lines 30-55), and visual elements (lines 160-236)"

### Linear Magic Words for Commits
**Closing Magic Words (Complete/Close Issues):**
- **close, closes, closed, closing**: Standard issue closure
- **fix, fixes, fixed, fixing**: For bug fixes
- **resolve, resolves, resolved, resolving**: For problem resolution
- **complete, completes, completed, completing**: For task completion

**Non-Closing Magic Words (Reference Without Closing):**
- **ref, refs, references**: Simple reference to issue
- **part of**: Partial contribution to issue
- **related to**: Related work
- **contributes to**: Contributing work
- **toward, towards**: Work progressing toward issue resolution


## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linting
npm run lint
```

## Project Architecture

This is a Next.js 13.5 application using the App Router that visualizes cybersecurity project portfolios on an interactive maturity curve for CISOs.

### Key Components

- **ProgressCurve.tsx**: Core visualization component that renders the interactive progress curve with SVG. Contains complex mathematical calculations for curve positioning and responsive design logic.
- **Project Data Model**: All project data is centralized in `lib/data.ts` with TypeScript interfaces defining project structure, timeline markers, and category classifications.

### Data Structure

Projects are categorized into four types:
- Unplanned Work (emergency/reactive)
- IS Projects (information security)
- IT/Business Projects (cross-functional)
- Business-as-Usual (operational)

Each project has:
- Position on curve (0-1 float)
- Risk level (L/M/H)
- Complexity rating (L/M/H)
- Timeline with visual markers (○●△▲⊗)

### Tech Stack

- **Frontend**: Next.js 13.5 + TypeScript + Tailwind CSS
- **Components**: Custom React components (ProgressCurve, Header, Sidebar)
- **Visualization**: Custom SVG rendering with responsive mathematics
- **Styling**: Tailwind with 8px spacing grid system

### Code Organization

```
app/                 # Next.js App Router
├── layout.tsx       # Root layout
├── page.tsx         # Main dashboard
└── globals.css      # Global styles

components/
├── features/        # Core business logic
├── layout/          # Layout components
└── ui/             # Reusable UI primitives

lib/
├── data.ts         # Project data and types
└── utils.ts        # Utility functions
```

## Customization

To add projects, edit `lib/data.ts` and add objects to the projects array with required fields: name, position, category, risk, complexity, timeline.

The application uses a design system with Apple-level aesthetics and follows responsive design patterns with mobile-first approach.

## Design System

### Color System

**Risk Level Colors:**
- High: `#fee2e2` (background), `#991b1b` (text)
- Medium: `#fef3c7` (background), `#92400e` (text)
- Low: `#dcfce7` (background), `#166534` (text)

**Complexity Level Colors:**
- High: `#fee2e2` (background), `#991b1b` (text)
- Medium: `#fef3c7` (background), `#92400e` (text)
- Low: `#dcfce7` (background), `#166534` (text)

### Spacing System

Following an 8px grid system:
- **Base padding**: 10px
- **Small breakpoint padding**: 12px
- **Large breakpoint padding**: 15px
- **General spacing**: Use multiples of 4px or 8px (4px, 8px, 16px, 24px, 32px, 48px)

### Typography

- **Tooltips**: Bold titles (`font-bold`), medium category text (`font-medium`)
- **Badges**: Semibold text (`font-semibold`)
- **Responsive sizing**: Defined in component-specific dimension configs (e.g., `TOOLTIP_DIMENSIONS`)

### Component Patterns

- **Border radius**: 8-10px for modern rounded corners
- **Shadows**: Subtle drop shadows using `rgba(0, 0, 0, 0.1)` with `blur(4px)`
- **SVG text positioning**: Use `dominantBaseline` attribute for consistent alignment

## Linear Issue Integration
- This project uses Linear for task management (issues like FOR-129, FOR-123, etc.)
- When committing changes that address Linear issues, use magic words in commit messages

### Closing Magic Words (Complete/Close Issues)
- **close, closes, closed, closing**: Standard issue closure
- **fix, fixes, fixed, fixing**: For bug fixes
- **resolve, resolves, resolved, resolving**: For problem resolution
- **complete, completes, completed, completing**: For task completion

### Non-Closing Magic Words (Reference Without Closing)
- **ref, refs, references**: Simple reference to issue
- **part of**: Partial contribution to issue
- **related to**: Related work
- **contributes to**: Contributing work
- **toward, towards**: Work progressing toward issue resolution

## Linear Best Practices

### Issue Title Management
- **NEVER include "FOR-##:" prefixes in Linear issue titles** - these are the project's internal numbering system
- Issue titles should be clean, professional descriptions without identifier prefixes
- Example: Use "Visual Framework Standardization" NOT "FOR-90: Visual Framework Standardization"
- When creating or updating issues, use descriptive titles that stand alone

### Comprehensive Issue Querying
- **Always search exhaustively** when asked to find or update Linear issues
- Use multiple search strategies to ensure complete coverage:
  1. Search by team ID with pagination: `mcp__Linear__list_issues` with `teamId` and `limit` parameters
  2. Search by project ID: `mcp__Linear__list_issues` with `projectId` parameter  
  3. Search by query terms: Use `query` parameter to find specific content
  4. For large result sets, use pagination with `after` parameter to get all results
- **Check backlog systematically** - don't assume first search captures everything
- When updating multiple issues, verify each change by fetching the issue afterward

### Issue Search Troubleshooting
- If search returns "exceeds maximum tokens" error, use smaller `limit` values (10-25)
- Use `after` parameter for pagination to get complete issue lists
- Combine multiple search approaches: team-based, project-based, and query-based
- Always verify team and project IDs are correct before searching

### Issue Update Verification
- After updating issue titles, fetch the issue again to confirm changes applied
- When asked to update "all issues with X", search comprehensively first, then update systematically
- Keep track of updated issues to avoid missing any in the batch

### Project Structure Understanding
- Project: "CISO Work Cycle"
- Team: "FS-ISAC Collab"
- Always filter by correct project when searching to avoid irrelevant results

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexfoley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
