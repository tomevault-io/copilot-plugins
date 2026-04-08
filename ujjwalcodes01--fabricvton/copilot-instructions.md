## fabricvton

> **fabricvton** is a B2B Shopify App Plugin that enables fashion/e-commerce brands to integrate a **“Virtual Try-On” (VTON)** button directly into their product pages. Built as a public Shopify app using **Theme App Extensions**, the plugin allows customers to upload their image and preview clothing virtually — without storing any photos.

# 🧩 Project: fabricvton — Virtual Try-On Shopify Plugin

## 📦 Overview

**fabricvton** is a B2B Shopify App Plugin that enables fashion/e-commerce brands to integrate a **“Virtual Try-On” (VTON)** button directly into their product pages. Built as a public Shopify app using **Theme App Extensions**, the plugin allows customers to upload their image and preview clothing virtually — without storing any photos.

This is a fully GDPR-compliant, privacy-first app. Merchants can install it from the Shopify App Store, configure it through the admin dashboard, and monitor try-on activity. All try-ons are processed via a third-party VTON model API (initially), with a transition to a custom model in later phases.

---

## 🧱 Architecture Overview

| Layer        | Tech Stack                            |
|--------------|----------------------------------------|
| Frontend     | React + Polaris (admin), JS (storefront) |
| Backend      | Node.js + Express                      |
| Model API    | Replicate / HuggingFace (initially)    |
| Auth         | OAuth 2.0 (token expires every 24h)    |
| Theme Ext    | Shopify App Embed Block (2.0 themes)   |
| Hosting      | GCP / Railway / Vercel (HTTPS)         |
| DB           | MongoDB or PostgreSQL (token storage)  |

---

## 🧪 Functionality by Role

### 🛍️ Customer View (on Merchant's Store)

- “Try-On” button embedded via Shopify **App Embed Block**
- Uploads their photo → model processes image → shows result
- No photo stored (stateless)

---

### 🧑‍💼 Admin Dashboard (for Merchants)

- Install plugin via Shopify App Store
- Enable/disable “Try-On” block via Theme Editor
- Upload garment images (optional)
- View usage stats and basic try-on analytics
- Monitor plan/billing

---

### 👑 Super Admin Dashboard (You)

- See all registered stores + API usage
- Approve/disable stores
- Monitor backend health + logs
- Trigger billing reports
- Launch model updates or A/B testing

---

## 📌 Core Modules

### 1. Shopify OAuth Flow
- Follows **2026+ OAuth flow** (no static token)
- Generates short-lived access tokens per shop
- Stores securely with `shop`, `expires_at`, and `scope`

### 2. Theme App Extension
- Injects Try-On button using **App Embed Block**
- Merchants can drag/drop into product template via Shopify Theme Editor

### 3. Try-On API Route
- Accepts user image + product info
- Sends to VTON model API (Replicate, etc.)
- Returns generated try-on image
- Ensures image is **not stored**

### 4. Responsive UI
- Fully mobile responsive Try-On modal
- Try-On button visible on all screen sizes
- Blur background when modal is open

---

## 🧭 Phased Development

### ✅ Phase 1 — MVP (Current)
- App install, OAuth, Try-On button
- External model API
- Fully deployed to `https://www.fabricvton.com`

### 🔜 Phase 2 — Admin + Billing
- Merchant dashboard, analytics, subscription plans

### 🔜 Phase 3 — Own Model
- Custom model hosted on GCP/Azure
- Switch from Replicate to in-house inference

---

## 📊 Monetization Strategy

- B2B SaaS — merchants pay per try-on or subscription
- Merchant installs app via Shopify
- Usage-based billing via Shopify Billing API
- Profit scales with client usage

---

## 🔐 Privacy & Compliance

- No customer photos are stored
- Complies with GDPR by design
- Stateless model inference
- Secure API token storage + HTTPS everywhere

---

## 🔗 Domain & Hosting

- Primary domain: `https://www.fabricvton.com`
- GCP preferred for hosting + autoscale GPU inference
- Shopify App Store listing pending (after Phase 2)

---

## ✉️ Contact / Maintenance

- Super Admin: [Your Name / Email]
- Support: support@fabricvton.com (to be configured)

---

## 🗂️ Repo Folder Structure (Suggested)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UjjwalCodes01)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/UjjwalCodes01)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
