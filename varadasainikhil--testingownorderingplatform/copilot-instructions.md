## testingownorderingplatform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a SwiftUI-based iOS ordering platform app using Firebase as the backend. The app allows users to browse menu items, add them to a cart, and manage their orders.

## Build and Development Commands

### Building and Running
- **Build**: `Cmd+B` in Xcode or use Product → Build
- **Run**: `Cmd+R` in Xcode or use Product → Run  
- **Clean Build Folder**: `Cmd+Shift+K` in Xcode

### Testing
- **Run Tests**: `Cmd+U` in Xcode or use Product → Test

## Architecture

### MVVM Pattern with SwiftUI
The app follows MVVM architecture using SwiftUI's `@Observable` macro:

- **Models**: `MenuItem` and `User` structs in `/Models/`
- **Views**: SwiftUI views in `/Views/` folder
- **ViewModels**: `@Observable` classes in `/ViewModel/` folder
- **Components**: Reusable UI components in `/Components/`

### Key Architectural Patterns

#### Environment-Based State Management
The app uses SwiftUI's environment system for shared state:
- `CartViewViewModel` is injected as environment object in `TestingOwnOrderingPlatformApp.swift`
- Views access it via `@Environment(CartViewViewModel.self) var cartViewModel`

#### Authentication Flow
- `MainViewViewModel` handles authentication state using Firebase Auth listeners
- Shows `LoginView` when not authenticated, `TabView` when authenticated
- Auth state listener is set up in `init()` and cleaned up in `stopHandler()`

#### Firebase Integration
- **Authentication**: Firebase Auth with email/password
- **Database**: Firestore for storing menu items and user cart data
- **Data Structure**:
  - `menuItems` collection: Contains all menu items
  - `users/{userId}/cart/current` document: Stores user's cart items

### Navigation Structure
```
MainView (auth gate)
├── LoginView (unauthenticated)
└── TabView (authenticated)
    ├── MenuView (Order tab)
    ├── CartView (Cart tab) 
    └── ProfileView (Profile tab)
```

## Firebase Data Model

### Collections
- **menuItems**: Menu items with fields: name, description, price, isEntree, isAppetizer, isVegetarian
- **users**: User documents containing cart subcollection

### Cart Data Structure
Cart items are stored as: `[String: Int]` where key is menuItemId and value is quantity

## Common Development Patterns

### Adding New Views
1. Create View in `/Views/` folder
2. Create corresponding ViewModel in `/ViewModel/` folder using `@Observable`
3. Import required Firebase modules (`FirebaseAuth`, `FirebaseFirestore`)
4. Follow existing naming convention: `[ViewName]View` and `[ViewName]ViewModel`

### Firestore Operations
- Use `@FirestoreQuery` for real-time data fetching (see `MenuView`)
- Use `Firestore.firestore()` instance for write operations
- Always check for authenticated user: `Auth.auth().currentUser?.uid`
- Use `setData(_:)` for dictionaries, `setData(from:)` for Codable objects

### Environment Object Usage
- Access cart functionality via: `@Environment(CartViewViewModel.self) var cartViewModel`
- Call methods like: `cartViewModel.addItemsToCart(menuItemId:count:)`

## Dependencies

The project uses Swift Package Manager with these key dependencies:
- Firebase iOS SDK (Auth, Firestore, Core)
- All Firebase dependencies are managed automatically via SPM

## Common Issues

### Firebase Integration
- Ensure `GoogleService-Info.plist` is properly added to the project
- Firebase is configured in `AppDelegate.application(_:didFinishLaunchingWithOptions:)`

### Cart State Management
- Cart items are persisted to Firestore in real-time
- Use `menuItem.id ?? menuItem.name` as fallback identifier for cart operations
- Cart operations should always call `updateCartItemsToFirebase()` after local updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/varadasainikhil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/varadasainikhil)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
