## stock-dash

> **Stock Competition Dashboard** (`stock-dash`) is a real-time, client-side React application built with Vite. It serves as a portfolio tracking dashboard for a friendly stock-picking competition (2026 Edition). Participants start with virtual capital ($10,000), and the dashboard visualizes their performance, rankings, and analytics based on live data fetched from a Google Sheet.

# Project Overview

**Stock Competition Dashboard** (`stock-dash`) is a real-time, client-side React application built with Vite. It serves as a portfolio tracking dashboard for a friendly stock-picking competition (2026 Edition). Participants start with virtual capital ($10,000), and the dashboard visualizes their performance, rankings, and analytics based on live data fetched from a Google Sheet.

## Key Technologies

- **UI Framework:** React 19
- **Build Tool:** Vite 7
- **Styling:** Tailwind CSS 3
- **Data Visualization:** Recharts
- **Icons:** Lucide React
- **Data Parsing:** PapaParse (for CSV processing)
- **External Data Source:** Google Sheets (Published as CSV)
- **External APIs:** Logo.dev (for stock ticker logos)

## Architecture

This is a purely client-side Single Page Application (SPA).

- **Entry Point:** `src/main.jsx` mounts the application.
- **Main Logic:** `src/App.jsx` handles data fetching, CSV parsing, state management (participants, rankings, stats), and renders the entire dashboard UI.
- **Data Flow:** The application fetches a published Google Sheet CSV directly in the client, parses it, calculates performance metrics (P&L, returns, day changes, rankings), and displays them through various functional UI sections (leaderboard, charts, stat cards).

## Building and Running

The project relies on `npm` for package management and script execution.

- **Install Dependencies:** `npm install`
- **Start Development Server:** `npm run dev`
- **Build for Production:** `npm run build`
- **Preview Production Build:** `npm run preview`
- **Run Linter:** `npm run lint`

## Development Conventions

- **Styling:** Utility-first styling with Tailwind CSS. The UI is designed to mimic a native iOS finance application. It supports a dynamic Light and Dark mode using Tailwind's `class` strategy (toggled via a `dark` class on the `<html>` element, driven by system preference or user override). Colors utilize native-feeling hex codes (e.g., `#000000` for dark backgrounds, `#1C1C1E` for dark cards, `#F2F2F7` for light backgrounds, `#34C759` for positive returns, `#FF3B30` for negative returns).
- **Component Structure:** The UI is currently built as a single, large component in `src/App.jsx`. It includes several inline functional components to maintain the iOS aesthetic (e.g., `StatCard`, `StockListItem`). Future development or refactoring could involve splitting this into smaller, separate files.
- **Configuration:** External URLs and API keys (such as the Google Sheets URL and Logo.dev API key) are currently hardcoded as constants at the top of `src/App.jsx`.
- **Responsive Design:** The layout relies on Tailwind CSS flexbox and grid utilities to provide a native-feeling mobile experience (such as horizontally scrolling widget rows that snap to the view) which scales up to a clean multi-column layout on desktop browsers. Component state (`isMobile`) is occasionally used to alter chart properties based on window width.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uzairansaruzi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/uzairansaruzi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
