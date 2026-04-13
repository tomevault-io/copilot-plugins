## gitreader

> **GitReader** is a web-based e-reader application built with **Laravel** and **React (Inertia.js)**. Its primary goal is to mimic the **GitHub User Interface** pixel-perfectly. This allows the user to read books (specifically EPUBs) in a workplace environment while appearing to read technical documentation or browse code repositories ("Stealth Mode").

# GitReader - The Stealth E-Reader

## 1. Project Overview
**GitReader** is a web-based e-reader application built with **Laravel** and **React (Inertia.js)**. Its primary goal is to mimic the **GitHub User Interface** pixel-perfectly. This allows the user to read books (specifically EPUBs) in a workplace environment while appearing to read technical documentation or browse code repositories ("Stealth Mode").

## 2. Core Metaphor (UI Mapping)
The application maps standard E-reader concepts to GitHub UI concepts:

| E-Reader Concept | GitHub UI Counterpart | Description |
| :--- | :--- | :--- |
| **Library (Home)** | **User Profile Page** | The main dashboard showing the user's "repositories" (books). The "Popular Repositories" or "Pinned" section displays current reads. |
| **Book** | **Repository** | Clicking a book opens a repository view. |
| **Table of Contents** | **File Browser** | The main view of the repo lists files. Each "file" represents a chapter of the book. |
| **Chapter** | **File View (Blob)** | Opening a file displays the chapter content. |
| **Reading History** | **Contribution Graph** | The green heatmap on the profile tracks reading activity (words read/chapters finished) instead of code commits. |
| **Author/Metadata** | **Repo Description** | Book metadata (Author, Year) is displayed in the About/Sidebar section of the repo. |

## 3. Functional Specifications

### A. Book Import & Management
*   **Format:** Support for **.epub** files.
*   **Ingestion:**
    *   **Upload:** UI interface to upload `.epub` files.
    *   **Local Scan:** System automatically scans a specific directory (e.g., `storage/app/library`) for new files.
*   **Processing:** On import, the system parses the EPUB, extracts metadata (cover, title, author), and splits the content into "Chapters" (stored in the database) to allow for quick loading and "file-like" navigation.

### B. Reading Experience
*   **Visual Style (Phase 1):** **Rendered Markdown Mode**. The text is displayed similarly to a `README.md` file on GitHub (clean typography, headers, white background, standard GitHub markdown CSS).
*   **Navigation:**
    *   Pagination is handled via "Next/Previous" file buttons (standard GitHub interface for navigating files).
    *   Breadcrumbs mimic the path (e.g., `author-name / book-title / chapter-1.md`).
*   **Progress:**
    *   **Auto-Save:** The system remembers the exact scroll position or paragraph of the last read chapter.
    *   **Status:** Repositories (Books) show a progress bar or "language stats" bar indicating % completed.

### C. "Stealth" Features
*   **Camouflage:** The UI must use exact GitHub colors, fonts (system-ui), and layout spacing.
*   **Contribution Graph:** Logic to calculate "contributions" based on reading sessions (e.g., 1 chapter read = 1 commit).
*   **Future Expansion:** A "Code View" switch that renders the text as code comments or string variables with syntax highlighting (green/gray text on dark background) for maximum stealth.

## 4. Technical Stack
*   **Backend:** Laravel 12.x
*   **Frontend:** React (via Inertia.js)
*   **Styling:** Tailwind CSS (configured with GitHub color palette).
*   **Database:** SQLite or MySQL (local).
*   **Key Libraries (To be determined):**
    *   PHP EPUB Parser (to extract text/html from .epub).
    *   HTML to Markdown converter (optional, or sanitize HTML to look like Markdown).

## 5. Database Schema (Draft)

*   `books`: `id`, `title`, `author`, `cover_image`, `file_path`, `total_chapters`, `created_at`
*   `chapters`: `id`, `book_id`, `title` (file name), `content` (text/html), `order_index`
*   `reading_progress`: `user_id`, `book_id`, `current_chapter_id`, `scroll_position`, `last_read_at`
*   `daily_activity`: `user_id`, `date`, `activity_count` (for the contribution graph)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wakchu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wakchu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
