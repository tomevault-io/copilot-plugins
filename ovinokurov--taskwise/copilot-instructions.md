## taskwise

> This is a full-stack AI-powered task management application named **TaskWise**. It is built with [Next.js](https://nextjs.org/) and [TypeScript](https://www.typescriptlang.org/), uses [PostgreSQL](https://www.postgresql.org/) for the database with [Prisma](https://www.prisma.io/) as the ORM, and is styled with [Tailwind CSS](https://tailwindcss.com/).

## Project Overview

This is a full-stack AI-powered task management application named **TaskWise**. It is built with [Next.js](https://nextjs.org/) and [TypeScript](https://www.typescriptlang.org/), uses [PostgreSQL](https://www.postgresql.org/) for the database with [Prisma](https://www.prisma.io/) as the ORM, and is styled with [Tailwind CSS](https://tailwindcss.com/).

The application allows users to manage tasks through a modern interface. The core features include:
- An interactive calendar for task visualization.
- AI-powered task creation from natural language.
- A chat interface to query tasks using natural language.
- AI-generated productivity analysis and reports.
- Full CRUD (Create, Read, Update, Delete) functionality for tasks.
- Task prioritization and status tracking.
- Light and dark mode support.

The AI functionalities are powered by the [OpenAI API](https://openai.com/api/).

## Building and Running

### Prerequisites
- Node.js (v18 or later)
- npm
- PostgreSQL

### Setup and Execution
1.  **Install Dependencies:**
    ```bash
    npm install
    ```
2.  **Database Setup:**
    - Ensure your PostgreSQL server is running.
    - Create a `.env` file from the `.env.example` file.
    - Configure the `DATABASE_URL` and `OPENAI_API_KEY` in the `.env` file.
3.  **Run Database Migrations:**
    ```bash
    npx prisma migrate dev
    ```
4.  **Run the Development Server:**
    ```bash
    npm run dev
    ```
    The application will be available at [http://localhost:3000](http://localhost:3000).

### Other Commands
-   **Build for Production:**
    ```bash
    npm run build
    ```
-   **Start Production Server:**
    ```bash
    npm run start
    ```
-   **Lint the Code:**
    ```bash
    npm run lint
    ```

## Development Conventions

### Code
- The project is written in **TypeScript**.
- The codebase is located in the `src` directory.
- **Next.js App Router** is used for routing. The pages and API routes are located in `src/app`.
- **React Server Components** and **Client Components** are used. Look for the `'use client';` directive at the top of files.
- Global layout is defined in `src/app/layout.tsx`.

### Styling
- **Tailwind CSS** is used for styling.
- The configuration is in `tailwind.config.ts`.
- Global styles are in `src/app/globals.css`.
- **`next-themes`** is used for light/dark mode, configured in `tailwind.config.ts` with `darkMode: "class"`.

### Database
- **Prisma** is the ORM.
- The database schema is defined in `prisma/schema.prisma`.
- Database migrations are managed by Prisma Migrate.

### API
- API routes are located in `src/app/api`.
- The project uses the OpenAI API for AI-powered features.

### Linting
- **ESLint** is used for linting. The configuration is in `eslint.config.mjs`.

## Detailed Breakdown

### Frontend (Pages)

-   **`src/app/page.tsx` (Home/Calendar View):**
    -   **Purpose:** The main landing page of the application.
    -   **Functionality:** Displays a full-screen calendar using `react-big-calendar`. It fetches all tasks and displays them as events on the calendar. Clicking an event navigates the user to the detailed view for that task.
    -   **Connection:** Fetches data from the `/api/events` endpoint. Navigates to `/tasks/[id]`.

-   **`src/app/tasks/active/page.tsx` (Active Tasks List):**
    -   **Purpose:** To show the user a list of their current and completed tasks.
    -   **Functionality:** Fetches all tasks and separates them into two lists: "Active Tasks" and "Completed Tasks". Users can mark an active task as complete directly from this page. It also contains a prominent button to create a new task.
    -   **Connection:** Fetches data from `/api/tasks`. Links to `/tasks/new` and `/tasks/[id]`. Updates task status via a `PATCH` request to `/api/tasks/[id]`.

-   **`src/app/tasks/new/page.tsx` (New Task Creation):**
    -   **Purpose:** To create a new task, with or without AI assistance.
    -   **Functionality:** Provides a form for creating a new task. As the user types a general idea into an input field, it sends a request to the backend to get AI-powered suggestions for the task's title, description, priority, and other details. The user can accept the suggestion to auto-fill the form or fill it out manually.
    -   **Connection:** Gets AI suggestions from `/api/suggest-task`. Creates a new task by sending a `POST` request to `/api/tasks`.

-   **`src/app/tasks/[id]/page.tsx` (Task Details View):**
    -   **Purpose:** To display all information about a single task.
    -   **Functionality:** Shows the full details of a task, including its title, description, priority, status, time estimate, and due date. It provides buttons to edit or delete the task.
    -   **Connection:** Fetches data from `/api/tasks/[id]`. Links to `/tasks/[id]/edit`. Deletes the task via a `DELETE` request to `/api/tasks/[id]`.

-   **`src/app/tasks/[id]/edit/page.tsx` (Edit Task Page):**
    -   **Purpose:** To modify an existing task.
    -   **Functionality:** Pre-fills a form with the existing details of a task. The user can edit any field and submit the form to update the task.
    -   **Connection:** Fetches initial data from `/api/tasks/[id]` and sends updated data via a `PATCH` request to the same endpoint.

-   **`src/app/reporting/page.tsx` (Productivity Reports):**
    -   **Purpose:** To give the user AI-generated insights into their productivity.
    -   **Functionality:** A user can click a button to generate a report. This triggers a backend process that analyzes the user's task history (from a log file) and uses AI to generate charts, key metrics, a textual summary, and actionable insights.
    -   **Connection:** Triggers the report generation by calling `/api/analyze-logs`.

### Backend (API Routes)

-   **`/api/tasks`:**
    -   `GET`: Retrieves all tasks from the database.
    -   `POST`: Creates a new task and logs the creation event to `analytics.log`.

-   **`/api/tasks/[id]`:**
    -   `GET`: Retrieves a single task by its ID.
    -   `PATCH`: Updates an existing task's details.
    -   `DELETE`: Deletes a task from the database.

-   **`/api/events`:**
    -   `GET`: Fetches all tasks and formats them into a structure that the calendar component can display.

-   **`/api/suggest-task`:**
    -   `POST`: Takes a user's text input and uses the OpenAI API to suggest a structured task (title, description, priority, etc.).

-   **`/api/analyze-logs`:**
    -   `GET`: Reads the `analytics.log` file, sends the content to the OpenAI API for analysis, and returns a comprehensive productivity report.

-   **`/api/chat-query`:**
    -   `POST`: Answers a user's natural language question about their tasks. It fetches data from the database and the log file, provides it to the OpenAI API as context, and returns the AI-generated answer.

### Components

-   **`src/components/ChatWindow.tsx`**: A floating chat window that allows users to interact with a conversational AI. It sends user messages to the `/api/chat-query` endpoint and displays the AI's response.
-   **`src/components/FloatingActionButton.tsx`**: A simple floating action button (FAB) used to trigger a primary action on a page, such as opening the task creation modal.
-   **`src/components/NavDropdown.tsx`**: A reusable dropdown menu component for the navigation bar, built with Headless UI.
-   **`src/components/RefForwardingLink.tsx`**: A wrapper component that forwards refs to the Next.js `Link` component, enabling its use with libraries like Headless UI.
-   **`src/components/ResponsiveNavbar.tsx`**: The main navigation bar for the application. It is responsive and uses a hamburger menu on mobile devices.
-   **`src/components/TaskCreationModal.tsx`**: A modal dialog for creating new tasks. It includes the same AI-powered suggestion functionality as the dedicated new task page.
-   **`src/components/ThemeProvider.tsx`**: A wrapper for `next-themes` to provide light and dark mode functionality.
-   **`src/components/ThemeSwitcher.tsx`**: A button component that allows users to toggle between light and dark themes.

### Database Schema (`prisma/schema.prisma`)

-   **`Task` model**: The central model in the application. It contains the following fields:
    -   `id`: Unique identifier (CUID).
    -   `title`: The task's title.
    -   `description`: An optional, more detailed description.
    -   `status`: The current status of the task (e.g., `TODO`, `IN_PROGRESS`, `COMPLETED`).
    -   `priority`: The task's priority level, defined by the `Priority` enum.
    -   `timeEstimate`: An integer representing the estimated time in minutes.
    -   `dueDate`: An optional due date and time.
    -   `createdAt`, `updatedAt`, `completedAt`: Timestamps for tracking the task's lifecycle.
-   **`Priority` enum**: Defines the set of possible priority levels: `LOW`, `MEDIUM`, `HIGH`, and `URGENT`.

### Data Flow and Connections

The application follows a standard client-server architecture with a Next.js frontend and backend.

1.  **Database:** At the core is a **PostgreSQL** database, with the schema managed by **Prisma**. The `Task` model is the central data structure.

2.  **API Layer:** The Next.js API routes provide the interface between the frontend and the database. They handle all CRUD operations for tasks and also contain the logic for interacting with the **OpenAI API**.

3.  **Frontend Pages:** The React components in the `src/app` directory render the user interface. They use standard `fetch` calls to interact with the API routes to get, create, update, and delete data.

4.  **AI Integration:** The AI features are encapsulated in specific API routes (`/api/suggest-task`, `/api/analyze-logs`, `/api/chat-query`). This keeps the AI logic separate from the frontend components and the core task management logic. The frontend simply calls these API routes and displays the results.

5.  **Logging:** Task creation and completion events are written to `analytics.log`. This file serves as the data source for the productivity analysis features, decoupling the reporting feature from the main database.

## Dependencies Analysis

### Core
-   **`next`**: The Next.js framework.
-   **`react`**, **`react-dom`**: The React library.
-   **`typescript`**: The TypeScript language.

### Database
-   **`@prisma/client`**: The Prisma client for database access.
-   **`prisma`** (dev): The Prisma CLI for migrations and other database operations.

### UI & Styling
-   **`tailwindcss`**: A utility-first CSS framework for styling.
-   **`@headlessui/react`**: A library of unstyled, fully accessible UI components.
-   **`next-themes`**: For managing light and dark themes.
-   **`react-chartjs-2`**, **`chart.js`**: For creating charts and graphs.
-   **`react-datepicker`**: A component for selecting dates and times.
-   **`react-markdown`**: For rendering Markdown content.

### AI
-   **`openai`**: The official Node.js library for the OpenAI API.

### Tooling
-   **`eslint`**, **`eslint-config-next`**: For code linting.
-   **`autoprefixer`**, **`@tailwindcss/postcss`**: For CSS processing.
-   **`next-swagger-doc`**, **`swagger-ui-react`**: For generating and displaying API documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ovinokurov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
