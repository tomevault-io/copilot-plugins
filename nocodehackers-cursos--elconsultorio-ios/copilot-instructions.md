## elconsultorio-ios

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"El Consultorio" is an iOS application built with SwiftUI for Jesús Terrés. The app provides a login system with email authentication through XANO and displays educational content in a swipeable slider format.

## Development Setup

This is an iOS project that requires Xcode to build and run. Open `ElConsultorio.xcodeproj` in Xcode to get started.

## Architecture

### Core Components
- **ElConsultorioApp.swift**: Main app entry point
- **ContentView.swift**: Root view that handles authentication state
- **AuthenticationManager.swift**: Handles user authentication with XANO (currently uses mock data)
- **Models.swift**: Data models and service for question slides
- **LoginView.swift**: Email-based login interface
- **SliderView.swift**: Main slider container with TabView
- **SlideCardView.swift**: Individual slide layout with question, answers, and links

### Key Features
- Email-only authentication system (validates against XANO backend)
- Swipeable slider interface using TabView with PageTabViewStyle
- Each slide contains:
  - Question at the top
  - Up to 2 answers in the middle
  - Optional link button at the bottom
  - Background image with gradient overlay for text readability

### Data Flow
- Authentication state managed by `AuthenticationManager` using `@StateObject`
- Slide data managed by `SlideDataService` with mock data
- Ready for XANO integration (endpoints to be provided)

### Mock Users
Current mock authentication accepts these emails:
- jesus@terresadvisory.com
- admin@elconsultorio.com
- demo@example.com

---
> Source: [Nocodehackers-Cursos/ElConsultorio-iOS](https://github.com/Nocodehackers-Cursos/ElConsultorio-iOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
