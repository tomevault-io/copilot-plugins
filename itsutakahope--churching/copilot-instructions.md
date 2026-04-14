## churching

> This project is a comprehensive church management application integrating two main modules: Purchase Request Management and Tithing/Donation Calculation. It provides an intuitive user interface and robust role-based access control (RBAC).

# Churching (Church Management System)

## Project Overview
This project is a comprehensive church management application integrating two main modules: Purchase Request Management and Tithing/Donation Calculation. It provides an intuitive user interface and robust role-based access control (RBAC).

**Key Features:**
*   **Purchase Request Management:** Create, edit, delete requests; multi-level account category management; transfer/refund management; PDF voucher generation; email notifications.
*   **Tithing/Donation System:** Tithing task creation and management; cash/check separation; category management; PDF report generation.
*   **User System:** Firebase Authentication integration; RBAC (admin, finance_staff, treasurer); dark/light mode support.

## Architecture & Technologies
*   **Frontend:** React 18, Vite, React Router DOM, Tailwind CSS, Axios, jsPDF.
*   **Backend:** Firebase Cloud Functions (Node.js 22), Express.js.
*   **Database & Auth:** Firebase Firestore, Firebase Authentication.
*   **Testing:** Vitest, React Testing Library, jsdom.

## Building and Running

1.  **Install Dependencies:**
    ```bash
    npm install
    cd functions && npm install && cd ..
    ```
2.  **Environment Setup:**
    Create a `.env.local` file in the root with your Firebase and Gmail API configurations (refer to `README.md` for the specific keys required).
3.  **Start Development Servers:**
    ```bash
    npm run dev
    ```
    This concurrently starts the Vite dev server (frontend) and Firebase Emulators (backend).
    *   Alternatively, run `npm run dev:client` for just the frontend, or `npm run emulate:functions` for just the backend.

## Testing
*   Run all tests once: `npm run test`
*   Run tests in watch mode: `npm run test:watch`
*   Open the testing UI dashboard: `npm run test:ui`

## Development Conventions
*   **Code Style:** Use ES6+ syntax, functional React components with Hooks.
*   **Styling:** Utilize Tailwind CSS utility classes for styling.
*   **Commits:** Follow semantic commit message guidelines (e.g., `feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `test:`).
*   **Security:** Ensure changes adhere to the defined Firestore security rules (`firestore.rules`) and respect the existing role-based access controls.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itsutakahope)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/itsutakahope)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
