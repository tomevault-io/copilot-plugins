## calsnap

> NexusTime is a lightweight, client-side Single Page Application (SPA) designed to solve the "availability tetris" problem. By ingesting standard `.ics` calendar files (common in Outlook, Google, Apple), it algorithmically determines optimal meeting slots. Unlike binary "free/busy" tools, NexusTime provides a fuzzy availability score, ranking time slots based on how many participants can attend. It features a clean, mobile-first UI and leverages Google's Gemini API to draft context-aware meeting invitations based on the selected slot.

# NexusTime: Intelligent Multi-Calendar Scheduler

## Abstract
NexusTime is a lightweight, client-side Single Page Application (SPA) designed to solve the "availability tetris" problem. By ingesting standard `.ics` calendar files (common in Outlook, Google, Apple), it algorithmically determines optimal meeting slots. Unlike binary "free/busy" tools, NexusTime provides a fuzzy availability score, ranking time slots based on how many participants can attend. It features a clean, mobile-first UI and leverages Google's Gemini API to draft context-aware meeting invitations based on the selected slot.

---

## Detailed Analysis & Architecture

### 1. Core Philosophy
*   **Privacy First:** All calendar parsing and logic happen in the browser. No calendar data is sent to a server.
*   **Resilience:** Handles imperfect data by offering "best possible" matches, not just perfect matches.
*   **Simplicity:** Drag-and-drop interface with immediate visual results.

### 2. Architecture
*   **Frontend Framework:** React 18 with TypeScript for type-safe logic.
*   **Styling:** Tailwind CSS for rapid, responsive, and aesthetic UI development.
*   **State Management:** React `useState` and `useMemo` for filtering and sorting slots efficiently.
*   **Services:**
    *   `IcsParser`: A regex-based service to extract `VEVENT` data (Start, End, Summary) from raw text files.
    *   `SchedulerEngine`: A time-bucket algorithm. It discretizes the timeline into 15-minute chunks, calculates a "heat map" of availability, and aggregates chunks into viable meeting slots.
    *   `GeminiService`: Utilizes `@google/genai` to generate professional meeting invites or analyze schedule conflicts.

### 3. Features
1.  **Multi-File Ingestion:** Accepts multiple `.ics` files, assigning a unique color/identity to each.
2.  **Dynamic Duration:** User can adjust meeting length (e.g., 30m, 60m), instantly recalculating available slots.
3.  **Availability Ranking:**
    *   *Tier 1 (Green):* 100% of calendars are free.
    *   *Tier 2 (Yellow):* >75% of calendars are free.
    *   *Tier 3 (Orange):* >50% of calendars are free.
4.  **Conflict Visualization:** Clearly shows *who* is missing in a partial match.
5.  **GenAI Invitation:** One-click generation of a meeting email draft using the chosen time and context.

### 4. Implementation Steps
1.  **Scaffolding:** Setup React DOM and Tailwind.
2.  **Data Layer:** Create strict TypeScript interfaces for `Calendar`, `Event`, and `TimeSlot`.
3.  **Parser Logic:** Implement a robust `.ics` reader that normalizes dates (handling UTC/Z timezones).
4.  **Scheduling Algorithm:** Implement the intersection logic to find and rank gaps.
5.  **UI Components:**
    *   `Uploader`: File input.
    *   `Controls`: Duration slider.
    *   `Results`: Masonry or Grid layout of time cards.
6.  **Integration:** Connect Gemini API for the "Draft Invite" feature.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/b2renger)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/b2renger)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
