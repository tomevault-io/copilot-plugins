## ukfr-smbc-app

> This is a Next.js application built with TypeScript, designed as a learning platform for the UK Financial Regulations (UKFR) exam. It uses Firebase for backend services like authentication and data synchronization. The app features study materials, mock exams, and progress tracking to help users prepare for their certification.


This is a Next.js application built with TypeScript, designed as a learning platform for the UK Financial Regulations (UKFR) exam. It uses Firebase for backend services like authentication and data synchronization. The app features study materials, mock exams, and progress tracking to help users prepare for their certification.

Key technologies include React for the frontend, Tailwind CSS for styling, and various libraries for data visualization and animation. It also integrates with a Gemini API for AI-powered features like keyword extraction from study materials.

The project is structured using the Next.js App Router, with clear separation of concerns for components, services, and utilities. Data for questions and study categories is stored as JSON files in the `public/data` directory. A number of scripts in the `scripts/` directory suggest a complex data pipeline for managing and translating the educational content.

## v2.0 Architecture

The application has undergone a significant refactoring to improve maintainability and scalability. The new architecture includes:

*   **Data Abstraction Layer**: A new `services/data` directory contains services that abstract the data persistence logic. The `HighlightService` encapsulates all CRUD operations for highlights, using both Firestore and LocalStorage.
*   **New Data Models**: The `Highlight` data model has been redesigned to use a robust anchor-based system instead of fragile offsets. The new data models are documented in `HIGHLIGHT_DATA_MODEL.md` and `CONFLICT_RESOLUTION_STRATEGY.md`.
*   **Component-based Viewer**: The material viewer has been broken down into smaller, more manageable components, such as `PdfRenderer`, `HtmlRenderer`, and `TableOfContents`.
*   **Testing**: The project now uses `vitest` for unit and component testing, and `cypress` for E2E testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kikuchi0790)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kikuchi0790)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
