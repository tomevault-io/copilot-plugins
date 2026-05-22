## goszakupki-auto-kp

> This project is a web-based automation tool designed to generate professional PDF commercial proposals (KP) by scraping data from the Belarusian government procurement portal (`goszakupki.by`). It streamlines the process of creating proposals for marketing and single-source procurement procedures and includes integration with Telegram for automatic document delivery.

# Goszakupki Auto KP (Commercial Proposal Generator)

## Project Overview

This project is a web-based automation tool designed to generate professional PDF commercial proposals (KP) by scraping data from the Belarusian government procurement portal (`goszakupki.by`). It streamlines the process of creating proposals for marketing and single-source procurement procedures and includes integration with Telegram for automatic document delivery.

**Key Features:**
*   **Automated Scraping:** Extracts organization details, lot descriptions, quantities, and other metadata from `goszakupki.by` pages.
*   **PDF Generation:** Generates A4 PDF documents using a customizable HTML template (`calculator-template.html`) and Puppeteer.
*   **Multi-Lot Support:** Handles procurements with single or multiple lots, allowing flexible selection and pricing for each.
*   **Telegram Integration:** Sends generated PDFs and procurement links directly to a specified Telegram chat/channel.
*   **Web Interface:** Provides a user-friendly frontend to input URLs, pricing, and description overrides.

## Tech Stack

*   **Runtime:** Node.js (>= 18.0.0)
*   **Server Framework:** Express.js
*   **Browser Automation:** Puppeteer (for scraping and PDF generation)
*   **Templating:** Handlebars
*   **Telegram API:** `node-telegram-bot-api`
*   **Configuration:** `dotenv`

## Architecture & Key Files

*   **`server.js`**: The main entry point. Sets up the Express server and API endpoints (`/generate`, `/send-to-telegram`, `/telegram-status`).
*   **`parser.js`**: Contains logic for scraping data from different types of `goszakupki.by` pages (Marketing, Single Source). Handles specific CSS selectors for data extraction.
*   **`pdfGenerator.js`**: Orchestrates the PDF creation process. Injects data into `calculator-template.html` and renders it to PDF using Puppeteer.
*   **`telegramSender.js`**: Manages the Telegram bot connection and message/file sending logic.
*   **`calculator-template.html`**: The Handlebars/HTML template used for the visual layout of the commercial proposal.
*   **`public/`**: Contains the frontend code (`index.html`, `manual.html`) serving the user interface.
*   **`generated/`**: Directory where generated PDF files are stored.
*   **`images/`**: Assets like logos and stamps used in the PDF template.

## Building and Running

### Prerequisites
*   Node.js v18+
*   npm

### Installation
```bash
npm install
```

### Configuration
Create a `.env` file in the root directory (see `.env.example` if available, or refer to `TELEGRAM_SETUP.md`):
```env
TELEGRAM_BOT_TOKEN=your_bot_token_here
```

### Development Scripts
*   **Start Server:**
    ```bash
    npm start
    ```
    Runs the application on the default port (3000).

*   **Development Mode:**
    ```bash
    npm run dev
    ```
    Runs the server with `nodemon` for hot reloading.

*   **Test Telegram:**
    ```bash
    npm run telegram:test
    ```
    Executes a specific test script for Telegram integration.

### Usage
1.  Start the server (`npm start`).
2.  Navigate to `http://localhost:3000` in your browser.
3.  Enter a `goszakupki.by` URL (e.g., `/marketing/view/...` or `/single-source/view/...`).
4.  Configure lot prices and selection.
5.  Generate the PDF and optionally send it to Telegram.

## Development Conventions

*   **Code Style:** Standard Node.js asynchronous patterns (`async/await`) are used throughout.
*   **Error Handling:** The server returns JSON responses with `success: false` and error messages for API failures.
*   **File Naming:** Generated PDFs follow a timestamped convention: `YYYY_MM_DD_HH_MM_SS_ID.pdf`.
*   **Versioning:** The project follows semantic versioning (currently v1.6.0).

---
> Source: [nbiotminsk/goszakupki_auto_KP](https://github.com/nbiotminsk/goszakupki_auto_KP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
