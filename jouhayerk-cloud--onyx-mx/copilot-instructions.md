## onyx-mx

> Onyx.mx is a comprehensive, web-based inventory and product management system designed for businesses involved in cross-border commerce, specifically between Mexico and the United States. The application facilitates the entire product lifecycle, from creation and acquisition to sales and shipping. It leverages modern web technologies, including React and Vite, and incorporates advanced features like 3D product visualization and AI-powered assistance through the Google Gemini API.

# Project: Onyx.mx

## Project Overview

Onyx.mx is a comprehensive, web-based inventory and product management system designed for businesses involved in cross-border commerce, specifically between Mexico and the United States. The application facilitates the entire product lifecycle, from creation and acquisition to sales and shipping. It leverages modern web technologies, including React and Vite, and incorporates advanced features like 3D product visualization and AI-powered assistance through the Google Gemini API.

The system is designed to be used by different stakeholders, including vendors, clients, and internal management, with role-based access control to tailor the user experience. The platform's focus on logistics is evident from its handling of essential export documents, making it a powerful tool for managing international supply chains.

## Building and Running

To run the application, follow these steps:

1.  **Install dependencies:**
    ```bash
    npm install
    ```

2.  **Set up environment variables:**
    Create a file named `.env.local` in the project root and add your Gemini API key:
    ```
    GEMINI_API_KEY=YOUR_GEMINI_API_KEY
    ```

3.  **Run the development server:**
    ```bash
    npm run dev
    ```

The application will be available at `http://localhost:3000`.

## Key Features

*   **Inventory Management:** Supports various methods for adding products, including fast entry, batch entry from spreadsheets, and video analysis for automated product creation.
*   **Product Catalog:** Provides a rich, interactive catalog with multiple views (e.g., inventory, market) and detailed product information panels.
*   **3D Visualization:** Integrates a `three.js`-based 3D viewer and workspace, allowing for detailed inspection and interaction with product models.
*   **Dashboard:** Offers a centralized view of key business metrics, including acquisitions, payments, and shipping status.
*   **User Roles:** Implements a role-based access control system to provide tailored experiences for different user types, such as administrators, vendors, and clients.
*   **AI-Powered Features:** Utilizes the Gemini API for various tasks, which could include product description generation, data analysis, and more.
*   **Customization:** Features a theming system and multi-language support (English and Spanish) to personalize the user experience.

## Logistics and Exportation (Mexico-USA)

Onyx.mx is specifically designed to streamline the exportation process from Mexico to the United States. The application manages and likely generates key shipping documents, such as:

*   **Certificate of Origin:** Essential for customs clearance and tariff calculations under trade agreements.
*   **Packing Lists:** Detailed documents that itemize the contents of each shipment.

The application's structure and data model are built to handle the complexities of international logistics, making it an invaluable tool for businesses operating in this corridor.

## Development Conventions

*   **State Management:** The project uses `jotai` for global state management. Atoms are defined in `src/lib/atoms.tsx`.
*   **Component Structure:** Components are organized by feature under `src/features`. Core components are in `src/components`.
*   **Styling:** The project will be migrated to a full Tailwind CSS integration with a custom theme defined in `tailwind.config.js`.
*   **API Integration:** The Gemini API is used, as indicated by the `@google/genai` dependency and the `GEMINI_API_KEY` environment variable.
*   **Routing:** The application will be migrated to use `react-router-dom` for routing.
*   **3D Visualization:** The project uses `three.js` for 3D rendering, with 3D-related components in `src/features/threed`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jouhayerk-cloud) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
