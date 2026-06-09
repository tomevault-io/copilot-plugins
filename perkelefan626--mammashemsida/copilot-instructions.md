## mammashemsida

> This project is a website for "Lucky Mystery Scoops," a retail concept centered around mystery scoops of charms and treasures. The website features a dual-theme design narrative ("Kuromi" for dark/mischievous and "My Melody" for light/sweet), bridging "mischievous gothic" and "soft sweetness."

# Project Overview: Lucky Mystery Scoops

This project is a website for "Lucky Mystery Scoops," a retail concept centered around mystery scoops of charms and treasures. The website features a dual-theme design narrative ("Kuromi" for dark/mischievous and "My Melody" for light/sweet), bridging "mischievous gothic" and "soft sweetness."

## Core Technologies
- **Frontend:** HTML5, Vanilla CSS3 (utilizing Glassmorphism and Tactile Modernism).
- **Design System:** Detailed specifications for colors, typography, and components found in the `DESIGN.md` file.

## Directory Structure
- `Assests/`: Contains project-wide SVG icons and package imagery. Note: This directory name is intentionally spelled `Assests`.
    - `Paket 1.png` to `Paket 4.png`: Mystery scoop package representations.
    - `heart_smile_...svg`: Favorite/Heart icon.
- `Main-Website/`: The primary active development directory.
    - `index.html`: Main landing page entry point with Kuromi and My Melody sections.
    - `checkout.html`: Functional checkout page with shipping forms and cart summary.
    - `style.css`: Comprehensive styling for the dual-theme experience (900+ lines).
    - `Logga v1.png`, `Logga v1 mörk.png`: Brand assets.
    - `Instagram.png`, `Tik-Tok.png`: Social media icons.
    - `shopping_cart_...svg`: Cart icon.
- `Mammas Hemsida prot 2/`: Prototypes and design references.
    - `DESIGN/DESIGN.md`: **Crucial** design specification document.
    - `Homepage/`, `Checkout 1/`, `Checkout 2/`: Reference code and screenshots for different site sections.

## Design Conventions
- **Themes:**
    - **Kuromi (Night-Mode):** Deep purple gradients, high-saturation pink accents, glassmorphism (background-blur). Implemented via a fixed background gradient on `body`.
    - **My Melody (Day-Mode):** Pastel pinks, pure white surfaces, soft ambient shadows.
- **Typography:**
    - **Headlines:** Spline Sans (Bold/Black).
    - **Body/UI:** Plus Jakarta Sans.
- **Components:**
    - **Buttons:** Pill-shaped, gradient (Kuromi) or solid pink (My Melody).
    - **Cards:** Rounded corners (1rem/16px), 24px internal padding.
    - **Toggles:** Heart-shaped checkboxes.

## Building and Running
Since this is a static frontend project:
- **Run:** Open `Main-Website/index.html` in any modern web browser.
- **Development:** Changes to `style.css` are reflected across both `index.html` and `checkout.html`.

## Contribution Guidelines
- Adhere strictly to the color palettes and typographic scales defined in `Mammas Hemsida prot 2/DESIGN/DESIGN.md`.
- Maintain the "Sparkle" motif through micro-interactions and decorative elements.
- Ensure all sections are responsive and utilize the 12-column fluid grid system.

---
> Source: [PerkeleFan626/MammasHemsida](https://github.com/PerkeleFan626/MammasHemsida) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
