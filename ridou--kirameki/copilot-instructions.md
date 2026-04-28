## kirameki

> This document outlines the technical requirements for the Kirameki website (`https://www.kirameki.earth/`). The website serves as the primary online presence for the Kirameki initiative, providing information about its mission, programs, and contact details, while also facilitating user interaction through forms and external links.

## Technical Product Requirements Document: Kirameki Website (kirameki.earth)

**1. Introduction**

This document outlines the technical requirements for the Kirameki website (`https://www.kirameki.earth/`). The website serves as the primary online presence for the Kirameki initiative, providing information about its mission, programs, and contact details, while also facilitating user interaction through forms and external links.

**2. Goals**

*   **Information Delivery:** Effectively present information about Kirameki's mission, programs, location, and team.
*   **User Engagement:** Enable users to inquire about programs, consultation, and booking.
*   **Brand Representation:** Reflect Kirameki's identity and values through design and content.
*   **Compliance:** Adhere to necessary web standards and regulations (e.g., cookie consent).
*   **Maintainability:** Allow for straightforward content updates and maintenance.

**3. Functional Requirements**

*   **3.1. Website Structure & Navigation:**
    *   **3.1.1. Core Pages:** The website must include the following distinct pages/sections, accessible via the primary navigation:
        *   Home
        *   Mission
        *   Programs
        *   Consultation
        *   Testimonies
        *   About Me
        *   Contact | Booking
    *   **3.1.2. Primary Navigation:** Implement a consistent navigation bar (header) across all pages, linking to the core pages listed in 3.1.1.
    *   **3.1.3. Footer Navigation:** Implement a footer containing:
        *   Quick Links (replicating primary navigation).
        *   Contact Information (Address, Phone/WhatsApp, Email).
        *   Legal Links: Imprint, Terms & Conditions, Privacy Policy, Cookie Policy.
        *   Copyright Information.
        *   Design Credit.
    *   **3.1.4. Internal Linking:** Implement contextual links within page content (e.g., "Explore Mission," "Explore Programs," "What Clients Say," "Contribute to Kirameki").
*   **3.2. Content Display:**
    *   **3.2.1. Text Content:** Display static text content as provided (headings, paragraphs, lists).
    *   **3.2.2. Image Content:** Display relevant images supporting the website's theme and content. Assume standard image optimization for web performance.
    *   **3.2.3. Branding Elements:** Display the Kirameki logo and name (including Japanese characters 煌めき) prominently.
*   **3.3. User Interaction:**
    *   **3.3.1. Contact/Booking Form:**
        *   Provide a form on the "Contact | Booking" page (or linked from it).
        *   Use `React Hook Form` for form handling and validation.
        *   The form should capture necessary user details (e.g., Name, Email, Subject, Message - *specific fields to be confirmed*).
        *   Upon submission, the form data should be sent to the backend (Node.js/Express) via `Axios` and potentially stored in the `Supabase` database via `Prisma`.
        *   Provide user feedback upon successful/failed submission.
        *   *(Integration Note: Consider if Calendly integration is desired here for direct booking instead of/in addition to a contact message).*
    *   **3.3.2. Social Media Links:** Include functional links to Kirameki's social media profiles:
        *   Instagram
        *   YouTube
        *   Threads
        *   TikTok
    *   **3.3.3. External Links:** Include functional links for:
        *   Crowdfunding Initiatives (linking to the relevant platform/page).
        *   *(Integration Note: If crowdfunding involves direct payments on-site, Stripe/Paypal integration will be needed).*
    *   **3.3.4. WhatsApp Link:** The phone number (`+81 80-2888-0788`) should ideally be formatted as a clickable link initiating a WhatsApp chat (`wa.me/` link).
    *   **3.3.5. Authentication (Optional - TBD):**
        *   If user accounts are required (e.g., for managing bookings, saving preferences), implement authentication using `Supabase Auth`.
        *   Support OAuth providers: Google, Facebook.
        *   Support custom OAuth provider: Instagram.
    *   **3.3.6. Booking & Payments (Optional - TBD):**
        *   Integrate `Calendly` for scheduling consultations or program bookings.
        *   Integrate `Stripe` and/or `Paypal` for processing payments related to programs, consultations, or donations/crowdfunding.
*   **3.4. Compliance:**
    *   **3.4.1. Cookie Consent Management:**
        *   Implement a cookie consent banner/widget upon the user's first visit.
        *   Allow users to accept, deny, or view/save preferences for different cookie categories (Functional, Preferences, Statistics, Marketing).
        *   Store user consent preferences.
        *   Provide a mechanism for users to manage/change their consent settings later (e.g., via a "Manage Consent" link/button, potentially in the footer).
    *   **3.4.2. Privacy Policy Access:** Ensure the Privacy Policy page is easily accessible (link in the footer).

**4. Non-Functional Requirements**

*   **4.1. Performance:**
    *   Optimize website assets (images, CSS, JS) for fast loading times.
    *   Target acceptable performance metrics (e.g., Google PageSpeed Insights scores).
*   **4.2. Responsiveness:**
    *   The website layout must adapt fluidly to various screen sizes and devices (desktops, tablets, mobile phones).
    *   Navigation must remain functional and usable on smaller screens (e.g., using a hamburger menu).
*   **4.3. Browser Compatibility:**
    *   Ensure consistent rendering and functionality across the latest versions of major web browsers (Chrome, Firefox, Safari, Edge).
*   **4.4. Security:**
    *   Serve the website over HTTPS.
    *   Implement basic security measures against common web vulnerabilities (e.g., cross-site scripting (XSS) if dynamic elements or user inputs are processed, though it seems mostly static).
    *   Ensure secure handling of data submitted via the contact form.
*   **4.5. Accessibility (A11y):**
    *   Adhere to basic web accessibility standards (WCAG AA where feasible). This includes:
        *   Semantic HTML structure.
        *   Alternative text for images.
        *   Keyboard navigability.
        *   Sufficient color contrast.
*   **4.6. Maintainability:**
    *   Structure the codebase (HTML, CSS, JS) logically for ease of updates.
    *   If using a Content Management System (CMS), it should allow non-technical users to update core content (text, images) easily. *Assumption: A CMS might be used, but the provided text doesn't confirm this.*
*   **4.7. SEO:**
    *   Implement basic on-page SEO elements: appropriate title tags, meta descriptions, header tags (H1, H2, etc.), and clean URLs.
    *   Leverage Next.js features for Server-Side Rendering (SSR) or Static Site Generation (SSG) to enhance SEO.

**5. Technology Stack**

*   **Frontend:**
    *   Framework: Next.js (React)
    *   Language: TypeScript
    *   Styling: Tailwind CSS
    *   State Management: Redux
    *   Forms: React Hook Form
*   **Backend:**
    *   Server: Node.js
    *   API Framework: Express
    *   HTTP Client: Axios
    *   Database: Supabase (PostgreSQL)
    *   ORM: Prisma
*   **APIs & Integrations:**
    *   Scheduling: Calendly
    *   Payments: Stripe, Paypal (*consider which is primary/secondary or if both are needed*)
    *   Authentication: Supabase Auth (Google, Facebook OAuth), Custom OAuth (Instagram)
*   **Hosting:** Vercel
*   **Tooling:**
    *   Code Format: Prettier
    *   Linting: ESLint (Next.js default configuration recommended)
    *   Version Control: Git, GitHub
    *   Unit Testing: Jest
*   **DevOps:** CI/CD Strategy via GitHub Actions integrated with Vercel (details TBD).

**6. Open Questions/Assumptions**

*   Are there specific fields required for the Contact/Booking form?
*   Is a specific CMS required or preferred?
*   What is the target platform for the Crowdfunding Initiatives link?
*   Are there specific performance targets (e.g., load time under X seconds)?
*   Confirmation of the exact functionality behind "Testimonies" (static display or dynamic system?). 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ridou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
