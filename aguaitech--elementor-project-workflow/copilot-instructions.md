## project-structure

> *   **Raw Page Config:** The raw page configurations downloaded from WordPress are stored in the `[pages/](mdc:pages)` folder. These files contain the full page data including metadata like page ID and title.

# Project Structure and Workflow

## Directory Structure

*   **Raw Page Config:** The raw page configurations downloaded from WordPress are stored in the `[pages/](mdc:pages)` folder. These files contain the full page data including metadata like page ID and title.
*   **Elementor Config:** The extracted and formatted Elementor configuration data is stored in the `[elementor/](mdc:elementor)` folder. Edits related to Elementor layouts should be applied to files in this directory.
*   **Compressed Elementor Config:** The compressed Elementor data, ready for upload, is stored in the `[compress/](mdc:compress)` folder.

All configuration files in these folders are named using the page slug and have a `.json` extension (e.g., `home.json`).

## Helper Scripts

There are two Node.js helper scripts for managing Elementor data:

1.  **`[compress_elementor.js](mdc:compress_elementor.js)`**:
    *   **Purpose:** Compresses the Elementor data from the `elementor/` folder into the `compress/` folder, making it suitable for uploading back to WordPress.
    *   **Usage:** `node compress_elementor.js [optional_arg]`
    *   **`optional_arg`:**
        *   _Not provided_: Compresses all files in `elementor/` and writes them to `compress/`.
        *   `skipExisting`: Compresses only the files in `elementor/` that do not already exist in `compress/`.
        *   `<filename.json>` (e.g., `home.json`): Compresses only the specified file from `elementor/` to `compress/`.
    *   **When to use:** Call this script *after* making changes to files in the `elementor/` folder and *before* updating the page on the WordPress site. The compressed file from `compress/` should be used for the update.

2.  **`[extract_elementor.js](mdc:extract_elementor.js)`**:
    *   **Purpose:** Extracts and formats the Elementor data from the raw page configuration files located in the `pages/` folder and saves it to the `elementor/` folder.
    *   **Usage:** `node extract_elementor.js [optional_arg]`
    *   **`optional_arg`:**
        *   _Not provided_: Extracts data from all files in `pages/` and writes them to `elementor/`.
        *   `skipExisting`: Extracts data only for pages in `pages/` that do not already have a corresponding file in `elementor/`.
        *   `<filename.json>` (e.g., `home.json`): Extracts data only from the specified file in `pages/` to `elementor/`.
    *   **When to use:** Call this script *after* downloading or fetching new page data into the `pages/` folder to prepare the Elementor data for editing.

## Workflow Summary

1.  **Fetch/Download:** Get page data from WordPress, save it in the `pages/` folder (e.g., `pages/about-us.json`).
2.  **Extract:** Run `node extract_elementor.js [page-slug.json]` to extract Elementor data into `elementor/` (e.g., `elementor/about-us.json`).
3.  **Edit:** Modify the Elementor JSON data in the `elementor/` file.
4.  **Compress:** Run `node compress_elementor.js [page-slug.json]` to create the compressed version in `compress/` (e.g., `compress/about-us.json`).
5.  **Update:** Use the appropriate tool to upload the compressed file from `compress/` back to WordPress. Use the corresponding file in `pages/` to retrieve metadata like `pageId` if needed.

---
> Source: [aguaitech/Elementor_Project_Workflow](https://github.com/aguaitech/Elementor_Project_Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
