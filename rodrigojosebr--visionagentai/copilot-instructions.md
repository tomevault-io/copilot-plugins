## visionagentai

> This project is an AI-powered image analysis tool designed to generate reliable descriptions for a list of images provided in a spreadsheet. It employs a multi-agent AI pattern to improve the accuracy of the descriptions and reduce the likelihood of AI "hallucinations."

# GEMINI Project Context

## Project Overview

This project is an AI-powered image analysis tool designed to generate reliable descriptions for a list of images provided in a spreadsheet. It employs a multi-agent AI pattern to improve the accuracy of the descriptions and reduce the likelihood of AI "hallucinations."

**Core Technologies:**
*   **Language:** JavaScript (Node.js)
*   **AI Model:** Google Gemini (via `@google/generative-ai`)
*   **Data Input:** `xlsx` for reading Excel files (`data.xlsx`)
*   **Configuration:** `dotenv` for managing environment variables (like API keys)

**Architecture:**

1.  **Data Input:** The script reads a list of image URLs from the `url` column in the `data.xlsx` spreadsheet.
2.  **Evaluation Phase:** For each image, it makes three parallel, independent calls to the Gemini API, asking each "evaluator agent" for a single-sentence description.
3.  **Judgment Phase:** The three descriptions are then passed to a "judge agent." This final call to the Gemini API instructs the model to analyze the three inputs, act as a judge, and produce a single, consensus-based description.

## Building and Running

### 1. Installation

Install the required Node.js dependencies. You can use either npm or yarn:

```bash
# Using npm
npm install

# Or using yarn
yarn install
```

### 2. Configuration

The project requires a Google Gemini API key to function.

1.  Create a file named `.env` in the root of the project.
2.  Add your API key to the `.env` file in the following format:

    ```
    API_KEY=your_google_gemini_api_key_here
    ```

### 3. Running the Application

Execute the main script using the npm `start` command:

```bash
npm start
```

The script will process the images from `data.xlsx` and output the results to the console.

### 4. Running Tests

Currently, no specific test suite is configured. The default test script will only return an error.

```bash
npm test
```

## Development Conventions

*   The core logic is self-contained within `index.js`.
*   Asynchronous operations are handled using `async/await`.
*   Environment variables are used for sensitive data like API keys, adhering to security best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodrigojosebr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
