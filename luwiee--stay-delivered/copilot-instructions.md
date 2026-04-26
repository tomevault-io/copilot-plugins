## stay-delivered

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Stay Delivered is a mobile application enabling golfers to order food, beverages, and golf accessories directly from their location on the golf course. The app facilitates real-time communication between golfers (consumers) and cart service staff (providers) with GPS tracking for delivery.

### Key Features
- **Menu-less inquiry system**: Direct verbal/text inquiry eliminates need for digital menus
- **Real-time messaging**: Direct communication between consumers and providers
- **GPS-based delivery**: Real-time position tracking for delivery to golfer's current location

## Tech Stack

### Mobile
- **Flutter** (Dart) - Cross-platform development for iOS and Android

### Backend (Firebase)
- **Firebase Authentication** - User registration/login
- **Cloud Firestore** - Real-time database for orders, messages, users, courses
- **Firebase Cloud Messaging (FCM)** - Push notifications
- **Firebase Cloud Functions** - Server-side logic
- **Firebase Storage** - Media storage

### Firestore Collections
- `users` - Consumer and provider profiles
- `courses` - Golf course information
- `orders` - Order records with status
- `conversations` - Chat messages
- `transactions` - Payment records

## Architecture

### User Types
- **Food Consumer (Golfer)**: Primary user, tracked via GPS, orders items
- **Food Provider (Cart Staff)**: Receives orders, has access to consumer locations, handles multiple conversations
- **Golf Course Administrator**: Future role for menu/pricing management

### Communication Flow
- Firestore real-time listeners for instant messaging
- FCM push notifications when app is backgrounded
- Real-time order status updates via Firestore subscriptions

## Development Guidelines

### Git Commits
- Commit regularly with descriptive messages
- Never mention AI/LLM assistance (Claude, ChatGPT, Copilot, etc.) in commit messages or code comments

### Working Style
- Never assume requirements or implementation details always check relevant code first
- Always ask clarifying questions before proceeding with ambiguous tasks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luwiee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
