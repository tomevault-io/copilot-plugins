## ebayimporter

> This document outlines the steps to create a converter from a Shopify CSV export to an eBay CSV import format.

# Gemini Plan: Shopify to eBay Converter

This document outlines the steps to create a converter from a Shopify CSV export to an eBay CSV import format.

## 1. Understand the Goal
The primary goal is to convert the `products_export_1.csv` file from Shopify into a new CSV file (`ebay_import.csv`) that is compatible with eBay's File Exchange format.

## 2. Analyze Input and Instructions
- **Shopify Export (`products_export_1.csv`):** This file contains the product data. Key columns to be used are: `Handle`, `Title`, `Body (HTML)`, `Vendor`, `Product Category`, `Type`, `Tags`, `Variant Price`, `Variant SKU`, `Variant Inventory Qty`, `Image Src`, and `Status`.
- **eBay Instructions (`Anleitungen` directory):** The PDFs in this directory describe the required format for the eBay import CSV. The mandatory fields are `*Action`, `*Category`, `*Title`, `*Description`, `*ConditionID`, `*Format`, `*Duration`, `*StartPrice`, `*Quantity`, `PicURL`, `*DispatchTimeMax`, and `*ReturnsAcceptedOption`.

## 3. Implementation Plan

### Step 3.1: Modify `main.py`
- **Comment out AI functionality:** The `improve_html_with_image` function call in the `main` function will be commented out as requested.
- **Read Shopify CSV:** The script will read the `products_export_1.csv` file.
- **Product Data Aggregation:** The script will iterate through the CSV and group all rows that belong to the same product (using the `Handle` column). It will collect all `Image Src` URLs for each product.
- **Create eBay CSV:** A new file named `ebay_import.csv` will be created with the correct eBay headers.
- **Column Mapping:** The script will map the Shopify data to the eBay format as follows:
    - `*Action`: Set to `Add` for all active products.
    - `*Category`: **Placeholder.** This is a critical field that needs a mapping from Shopify's category (`Product Category` or `Type`) to eBay's numerical category IDs. For the initial version, a placeholder value (e.g., `12345`) will be used. This will need to be manually updated later or a mapping solution will need to be implemented.
    - `*Title`: Mapped from `Title`.
    - `*Description`: Mapped from `Body (HTML)`.
    - `*ConditionID`: Set to `1000` (New) for all products, assuming they are new.
    - `*Format`: Set to `FixedPrice`.
    - `*Duration`: Set to `GTC` (Good 'Til Cancelled).
    - `*StartPrice`: Mapped from `Variant Price`.
    - `*Quantity`: Mapped from `Variant Inventory Qty`.
    - `PicURL`: All `Image Src` URLs for a product will be concatenated, separated by `|`.
    - `*DispatchTimeMax`: Set to a default value, e.g., `3` (days).
    - `*ReturnsAcceptedOption`: Set to `ReturnsAccepted`.
    - `CustomLabel`: Mapped from `Variant SKU` or `Handle` for internal tracking.
    - `Product:Brand`: Mapped from `Vendor`.
- **Filtering:** Only products with `Status` as `active` will be processed.

### Step 3.2: Future Improvements (Optional)
- Implement a robust category mapping solution. This could involve creating a separate mapping file (e.g., a CSV or JSON) or using an API to find the best eBay category.
- Add more optional fields to the eBay CSV for a richer listing, such as `ItemSpecifics`.

## 4. Execution
- Run the modified `main.py` script.
- The output will be the `ebay_import.csv` file, ready for review and potential upload to eBay.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/susesKaninchen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/susesKaninchen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
