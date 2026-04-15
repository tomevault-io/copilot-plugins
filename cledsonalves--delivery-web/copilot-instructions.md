## delivery-web

> This is a React TypeScript web application named "jarinu-shop", likely functioning as an e-commerce or delivery platform. It integrates with Google's Generative AI (Gemini) and Mercado Pago for payment processing. The application uses a custom API for backend communication, hosted at `https://api.jarifast.com.br`.

# Project Overview

This is a React TypeScript web application named "jarinu-shop", likely functioning as an e-commerce or delivery platform. It integrates with Google's Generative AI (Gemini) and Mercado Pago for payment processing. The application uses a custom API for backend communication, hosted at `https://api.jarifast.com.br`.

## Technologies Used

*   **Frontend:** React, TypeScript
*   **Build Tool:** Vite
*   **Routing:** React Router DOM (HashRouter)
*   **State Management:** React Context API (Auth, City, Cart)
*   **API Communication:** Custom `fetch`-based API service
*   **AI Integration:** Google Gemini (`@google/genai`)
*   **Payment Gateway:** Mercado Pago (`@mercadopago/sdk-react`)

## Building and Running

### Prerequisites

*   Node.js (LTS version recommended)

### Local Development

1.  **Install Dependencies:**
    ```bash
    npm install
    ```
2.  **Configure Gemini API Key:**
    Set your `GEMINI_API_KEY` in a `.env.local` file in the project root.
    ```
    GEMINI_API_KEY=YOUR_GEMINI_API_KEY
    ```
3.  **Run the Development Server:**
    ```bash
    npm run dev
    ```
    The application will typically be available at `http://localhost:3000`.

### Building for Production

```bash
npm run build
```
This command will compile the application into the `dist` directory.

### Deploying to GitHub Pages

This project is configured to deploy to GitHub Pages using `gh-pages`.
```bash
npm run deploy
```

### Previewing Production Build Locally

```bash
npm run preview
```

## Development Conventions

*   **Code Style:** Follows standard React and TypeScript best practices.
*   **Component Structure:** Components are organized in the `components/` directory.
*   **Page Structure:** Main application views are located in the `pages/` directory.
*   **Context API:** Global state is managed using React's Context API, with providers located in the `context/` directory.
*   **API Services:** Backend interactions are centralized in the `services/` directory, using a custom `apiRequest` utility.
*   **Environment Variables:** Sensitive information like API keys are managed via environment variables, loaded by Vite.

## Key Files and Directories

*   `App.tsx`: Main application component, handles routing and context providers.
*   `index.html`: Entry point for the web application.
*   `package.json`: Project metadata, dependencies, and scripts.
*   `vite.config.ts`: Vite build configuration.
*   `tsconfig.json`: TypeScript configuration.
*   `components/`: Reusable React components.
*   `context/`: React Context API providers for global state.
*   `pages/`: Top-level application views/pages.
*   `services/`: API integration and other service-related logic.
*   `.env`, `.env.development`: Environment variable configuration files.
*   `API.md`: Likely contains documentation for the backend API.
*   `SHOPEE_AFFILIATES.md`: Documentation related to Shopee affiliates.
*   `metadata.json`: Project metadata.
*   `types.ts`: Global TypeScript type definitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cledsonAlves)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cledsonAlves)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
