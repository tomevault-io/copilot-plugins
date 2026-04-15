## velokhaya-website

> **Reference Document:** [Project-Background.md](Project-Background.md)

# Velokhaya Life Cycling Academy - Project Context & Agent Guide

## 1. Project Overview & Source of Truth
**Reference Document:** [Project-Background.md](Project-Background.md)  
*Always refer to the Project Background for the definitive narrative, historical facts, and strategic audit.*

**Mission:** To use the bicycle as a vehicle for social transformation ("Life Cycling").  
**Core Metaphor:** "The Peloton" – A protective social structure that shields vulnerable youth from the "headwinds" of township life (crime, drugs, poverty).  
**Slogan:** "Cycling with a Purpose".

## 2. Brand Identity & Design System

### Color Palette
The brand uses a bold, primary color scheme reflecting energy and South African heritage.
*   **Brand Red:** `#E31E24` (Primary Action / Passion / Nutrition Pillar)
*   **Brand Yellow:** `#FFD700` (Highlight / Success / Sport Pillar)
*   **Brand Blue:** `#0054A6` (Trust / Depth / Education Pillar)
*   **Brand Black:** `#000000` (Backgrounds / Sophistication)
*   **Brand Dark:** `#171717` (Secondary Backgrounds)
*   **Brand Gray:** `#262626` (Cards / UI Elements)

### Typography
*   **Font Family:** 'Inter', sans-serif.
*   **Style:** Clean, modern, high-readability. Bold weights (700-900) used for impact headings.

### Visual Tone
*   **Do:** High-performance, athletic, resilient, professional, "Hero's Journey".
*   **Don't:** Pity, poverty porn, chaotic, "dusty charity".
*   **Key Imagery:** Cyclists in formation (peloton), blurred motion (speed), intense focus, academic success (graduates).

## 3. Target Audience & User Personas

1.  **The Corporate Sponsor (CSI/B-BBEE Manager)**
    *   **Goal:** Find a reliable, high-impact partner for Corporate Social Investment.
    *   **Needs:** Governance proof, B-BBEE alignment, clear impact metrics (matric pass rates), brand visibility.
    *   **Key Pages:** Impact, Support (Corporate Partner tier).

2.  **The Global Cyclist (UK/International Donor)**
    *   **Goal:** Support the sport they love in a meaningful way.
    *   **Needs:** Emotional connection ("The Peloton"), tax efficiency (Gift Aid/Buffalo Foundation), ease of payment.
    *   **Key Pages:** Support (International Donors), DNA (Hall of Fame).

3.  **The Local Enthusiast (Event Participant)**
    *   **Goal:** Join a ride or support a specific campaign.
    *   **Needs:** Event details (Spinathon), "cool factor", community belonging.
    *   **Key Pages:** Support (Current Campaigns), Home.

## 4. SEO & GEO Optimization Strategy

### Primary Keywords (SEO)
*   "Cycling Academy Cape Town"
*   "Velokhaya Khayelitsha"
*   "Sports Development NGO South Africa"
*   "Charity Cycling Team"
*   "Bicycle Empowerment Network"
*   "Youth Development Khayelitsha"

### Local Relevance (GEO)
*   **Location:** Khayelitsha, Cape Town, Western Cape, South Africa.
*   **Landmarks:** Khayelitsha Stadium, Phendula Crescent.
*   **Context:** Emphasize the "Township Economy" and "Township Sport" to capture local search intent.

### Semantic Entities
*   **People:** Sipho Mona (GM), Nicholas Dlamini (Olympian), Luthando Kaka (Captain), Anita Zenani (BMX Champion).
*   **Partners:** Pick n Pay, Matrix Fitness, MTN, Microsoft.
*   **Concepts:** "Life Cycling", "Safe Cycling", "No Homework No Ride".

## 5. Technical Architecture

*   **Stack:** Static HTML5.
*   **Styling:** Tailwind CSS v4 (compiled via CLI).
*   **Icons:** Lucide Icons (via CDN).
*   **Structure:**
    *   `index.html`: Hero, Hook, Partners.
    *   `our-approach.html`: History, Leadership, Hall of Fame.
    *   `impact.html`: The 3 Pillars (Sport, Education, Nutrition).
    *   `support.html`: Donation Tiers, Campaigns, International Giving.
    *   `contact.html`: Physical address, form, specific email contacts.

### ⚠️ Build Process (IMPORTANT)
After making any changes to HTML files that add or modify Tailwind CSS classes, you **MUST** rebuild the CSS:

```bash
npm run build
```

This compiles `src/input.css` → `css/styles.css`. Without this step, new Tailwind classes will not be applied and the page will appear broken.

**Files:**
*   Source: `src/input.css`
*   Output: `css/styles.css` (do not edit directly)

## 6. Content Strategy Guidelines

*   **The "Life Cycling" Pillars:** Always categorize impact into:
    1.  **Sport (The Carrot):** Medals, podiums, discipline.
    2.  **Education (The Purpose):** ICT lab, tutoring, matric results.
    3.  **Nutrition (The Fuel):** Daily meals, health.
*   **The "Container" Narrative:** Frame the shipping containers not as poverty, but as the "incubator" of talent.
*   **Call to Action (CTA):** Use active, sports-related verbs: "Join the Peloton", "Fuel the Future", "Ride with Us".

## 7. Critical Data Points (For Verification)
*   **Bank:** Standard Bank
*   **Account Name:** Velokhaya Cycling Academy
*   **UK Partner:** The Buffalo Foundation (Charity No. 1165676)
*   **Key Email:** `sipho@velokhaya.com` (Partnerships)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ginccc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ginccc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
