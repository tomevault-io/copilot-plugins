## prize-dictionary

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Prize Dictionary is a lottery prize reveal web application that uses QR codes and UUIDs to securely display prize names. The application prevents users from enumerating all available prizes by using unique identifiers instead of sequential numbers. It is built with vanilla HTML/JavaScript and hosted on GitHub Pages at https://kazfuku.github.io/prize-dictionary/.

## Running the Application

Open `index.html` directly in a browser. No build process, server, or dependencies are required.

## Architecture

The application consists of three HTML files:

### `index.html` (Entry Point)
- Simple redirect page that automatically forwards users to `list.html` after 1 second
- Provides manual link if automatic redirect fails
- No product data or business logic

### `list.html` (Prize List Page)
- Displays all available prizes with QR codes
- **Product Data**: Hardcoded array of product objects with `number`, `uuid`, and `name` properties
- **QR Code Generation**: Uses QRServer API (https://api.qrserver.com/v1/create-qr-code/) to generate QR codes
- Each prize shows:
  - Prize number (for administrative reference)
  - QR code linking to the prize detail page with UUID parameter
  - Direct link for non-QR access
- **Base URL**: GitHub Pages URL (https://kazfuku.github.io/prize-dictionary) for QR code generation

### `prize.html` (Prize Detail Page)
- UUID-based prize reveal page
- Accepts `id` parameter in URL query string (e.g., `prize.html?id={uuid}`)
- **Product Data**: Same hardcoded array as `list.html` with `number`, `uuid`, and `name` properties
- **UI Components**:
  - "賞品を表示" (Reveal Prize) button
  - Result display area (initially shows "???" placeholder)
- **Animation System**:
  - Lottery-style loading animation with bouncing dots (2-second delay for dramatic effect)
  - Confetti celebration animation that triggers on successful prize reveal
  - Fade-in transitions for result display
- **Security**: Prize name is only displayed after UUID validation and button click
- Displays error messages if UUID is missing or invalid

## User Workflow

1. User draws a lottery number
2. Administrator displays the corresponding prize's QR code from `list.html`
3. User scans QR code to access `prize.html?id={uuid}`
4. User clicks "賞品を表示" button to reveal prize name
5. Prize name appears with confetti animation

## Modifying Product Data

To add/modify products, update the `products` array in **both** `list.html` and `prize.html`. Each product object must have:
- `number`: Integer product ID (for administrative reference)
- `uuid`: UUID v4 string (unique identifier for secure access)
- `name`: String product name (in Japanese)

Important: Keep the product data synchronized between `list.html` and `prize.html`.

## Security Features

- UUIDs prevent sequential enumeration of prizes
- Prize names are hidden until user explicitly reveals them
- No direct number-to-prize mapping exposed in URLs
- QR codes provide secure, one-way access to prize details

## Deployment

Changes pushed to the `main` branch are automatically deployed to GitHub Pages. The site is configured to serve from the repository root.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kazfuku) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
