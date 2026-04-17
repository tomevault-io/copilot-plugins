## vidmarkect

> This project is a high-fidelity, production-ready **Laravel 11** application migrated from 63 standalone prototypes into a hardened cinematic video streaming marketplace. It features a robust dual-monetization engine, a high-performance discovery system, and multi-tenant creator tools.

# VidMarket - Video Platform Prototype

## Project Overview
This project is a high-fidelity, production-ready **Laravel 11** application migrated from 63 standalone prototypes into a hardened cinematic video streaming marketplace. It features a robust dual-monetization engine, a high-performance discovery system, and multi-tenant creator tools.

### Core Technologies (Current)
*   **Backend:** Laravel 11 (PHP 8.4)
*   **Frontend:** Laravel Blade + Tailwind CSS (Vite) + Alpine.js
*   **Database:** SQLite (Testing) / MySQL (XAMPP/Production) with complex relationship mapping and performance caching
*   **Storage:** Cloudinary (Private/Authenticated Video Delivery + Public Thumbnails)
*   **Payments:** Razorpay Integration (Secure INR Transactions & Subscriptions)
*   **Security:** Role-Based Access Control (RBAC), Custom Middleware, and Pessimistic Database Locking

### Architecture Highlights
*   **Intelligent Marketplace:** High-performance discovery with query caching (`Cache::remember`), advanced category/price filtering, and Netflix-style smart hover previews.
*   **Dual-Monetization Engine:** 
    *   **Wallet:** Internal coin-based economy with negative-ledger accounting, atomic tipping, and modal-driven refund requests.
    *   **Atomic Exchanges:** Secure video unlock logic using row-level locking (`lockForUpdate`) to prevent balance race conditions.
*   **Cloudinary Integration:** Optimized for security with **Authenticated Signed URLs** (2-hour expiry) for private video delivery and automated asset cleanup via shared `DeleteVideoAction`.
*   **Razorpay Ecosystem:** Fully integrated payment flow with server-side signature verification and order-amount validation to prevent client-side manipulation.
*   **Subscriptions:** Tiered Pro plans (Weekly/Monthly/Yearly) protected by `SubscriptionMiddleware`, supporting both active and trialing states.
*   **Creator Ecosystem:** Dedicated Studio for content management, automated earnings tracking (Tips/Sales), and rich public-facing brand profiles.
*   **Sentinel Guard (Security):** Admin suite for real-time traffic anomaly detection, user flagging, IP blacklisting, support ticket management, and mass-assignment protection.

## Recent Improvements (March 2026)
1.  **Reusable Components:** Created `video-card` and `video-card-row` Blade components for consistent video display across all pages.
2.  **Enhanced Discovery:** Added trending videos, free videos sections, and improved caching strategy.
3.  **UI/UX Improvements:** 
    - Unified styling across all pages
    - Better mobile responsiveness
    - Improved hover effects and transitions
    - SEO meta tags and favicon added
4.  **Bug Fixes:**
    - User model: Added `role` and `coin_balance` to fillable array
    - Fixed duplicate Material Symbols font link
    - Fixed ReviewController to check for ownership/pro subscription
    - Fixed middleware null user handling
    - Fixed wallet shop extra closing div
    - Fixed transaction icons and labels in wallet
5.  **Performance:** 
    - Improved eager loading with `with('creator')` 
    - Added more cache keys for video-related operations
    - Lazy loading on images

## Directory Structure
*   `vidmarket/`: The active Laravel application.
*   `stitch_video_player/`: Original high-fidelity HTML/CSS prototypes (Source of Truth for UI).

## Implementation Progress
The platform is **Complete, Optimized, and Architecturally Hardened**.

### Key Modules Implemented
1.  **Unified Dark Auth & Profiles:** Full Laravel Breeze integration with custom roles and a cinematic dark theme. Mass-assignment protected (`coin_balance` and `role`).
2.  **Video Management:** Refactored into Action classes (`StoreVideoAction`, `DeleteVideoAction`) for consistent handling of DB records and Cloudinary assets.
3.  **Transactional Core:** Secure coin-balance protection, dual-sided tipping records, and creator credit logic with deadlock prevention.
4.  **Responsive UI:** Mobile-friendly sidebar with Alpine.js state management and a unified global footer across all layouts.
5.  **Social & Support:** 5-star review engine with automated rating cache invalidation and a notification hub.
6.  **Polished Experience:** Custom cinematic error pages (404, 403, 500) and an enhanced Pro Exclusive portal.

## Development Conventions
*   **Surgical Aesthetics:** Maintain the high-contrast dark theme with primary accent `#8c25f4`.
*   **Performance First:** Use eager-loading and `isPro()` caching to minimize database overhead.
*   **Security First:** Server-side verification for all financial transactions and strictly enforced ownership checks on all personal data.

## Verification
The system is validated by a suite of **26 automated tests (64 assertions)** covering:
*   Secure Authentication & Custom Registration
*   Profile, Security, & Account Settings Updates
*   Role-Based Access Control (Admin/Creator/User)
*   Subscription Guard & Exclusive Content Logic
*   Transactional Integrity & Atomic Coin Transfers
*   Public Video Accessibility & Null-Safety

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bad-boy-01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
