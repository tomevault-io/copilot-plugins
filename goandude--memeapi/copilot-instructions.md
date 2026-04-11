## memeapi

> This project is a high-performance, serverless API that provides a random, context-specific meme image URL to client applications for use as a loading or transition screen. The primary goal is to provide a delightful and relevant user experience during wait times.

# Project Context: Contextual Meme Loading API

This project is a high-performance, serverless API that provides a random, context-specific meme image URL to client applications for use as a loading or transition screen. The primary goal is to provide a delightful and relevant user experience during wait times.

## 1. Project Goal and Architecture

* **Core Goal:** Serve a JSON payload containing a meme image URL based on a required query parameter: `context`.
* **Target Audience:** Web and mobile developers who need entertaining, dynamic loading indicators.
* **Architecture:** Serverless Function (Netlify/Vercel) + Database/Storage (Supabase). This ensures high scalability, low maintenance, and fast cold-start times.

## 2. Tech Stack and Frameworks

* **Serverless Platform:** Netlify Functions or Vercel Serverless Functions.
* **Language:** JavaScript (Node.js).
* **Database & Storage:** Supabase (PostgreSQL for metadata, Supabase Storage for images/GIFs).
* **Key Libraries:** `@supabase/supabase-js` for database and storage interaction.

## 3. Implementation Requirements

### A. Environment Variables
The function **MUST** use the following environment variables, which are configured in the deployment environment (Vercel/Netlify):
* `SUPABASE_URL`
* `SUPABASE_ANON_KEY`

### B. API Endpoint
* **Method:** `GET`
* **Path:** `/api/get-meme` (or the serverless path: `/.netlify/functions/get-meme`)
* **Query Parameter:** `context` (e.g., `?context=data_crunch`)
* **Default Behavior:** If `context` is missing, default to the category `'page_load'`.
* **CORS:** Responses **MUST** include the `Access-Control-Allow-Origin: *` header.

### C. Database Interaction
* The function will call the **Supabase RPC (Remote Procedure Call)** named `get_random_meme` and pass the `context` string as an argument.
* The function will then generate the **full public image URL** using the path returned by the RPC and the configured public storage bucket, which is named `memes`.

### D. Data Structure (Supabase)
The database table is named `memes`.
* **Table:** `memes`
* **Key Columns:**
    * `path` (TEXT): The relative path to the image in the Supabase Storage bucket.
    * `category` (TEXT): The tag used for filtering (e.g., 'page\_load', 'checkout\_payment', 'search\_query').

## 4. Key Files to Create

The Gemini CLI should focus on generating the following files:

1.  **`package.json`**: Node.js dependencies (`@supabase/supabase-js`, `dotenv` for local testing).
2.  **`get-meme.js`**: The primary serverless function logic (using the code previously provided).
3.  **`netlify.toml`** (if deploying on Netlify): Configuration to define the functions directory.
4.  **`supabase_functions.sql`**: The SQL code to create the `memes` table and the `get_random_meme` function.

## 5. Coding Conventions

* Use ES6 syntax (async/await, `import` statements, `const`/`let`).
* Ensure the code is defensive, with comprehensive `try...catch` blocks for all database and external calls.
* Prioritize clarity and directness in the response JSON, returning only the essential data (`url`, `category`).

---

I recommend starting with the `package.json` and then the `get-meme.js` file, following the plan outlined here.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goandude)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/goandude)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
