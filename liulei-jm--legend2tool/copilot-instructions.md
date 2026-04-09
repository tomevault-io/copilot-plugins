## legend2tool

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WPF application for configuring Legend of Mir (传奇) game servers. The application provides tools for:
1. Port configuration
2. Dynamic monster spawning configuration
3. Script optimization

The application is built with .NET 8.0 and uses WPF for the UI, with the following key technologies:
- CommunityToolkit.Mvvm for MVVM pattern implementation
- HandyControl for UI components
- IniFileParserStandard for configuration file parsing
- Serilog for logging
- Microsoft.Extensions.DependencyInjection for dependency injection

## Code Architecture

### Structure
- `App.xaml.cs`: Application entry point with DI container setup
- `MainWindow.xaml`: Main window with tabbed interface
- `ViewModels/`: Contains MVVM view models for each feature
- `Views/`: Contains WPF user controls for each feature
- `Services/`: Business logic and file operations
- `State/`: Application state management
- `Models/`: Data models for configuration files
- `Enums/`: Enumeration types
- `Messages/`: Messenger pattern classes for communication between components

### Key Patterns
1. **MVVM with CommunityToolkit**: Uses ObservableProperty attributes and IRecipient interfaces
2. **Dependency Injection**: Services registered in App.xaml.cs
3. **Messenger Pattern**: For communication between view models
4. **State Management**: ConfigStore holds application state

## Common Development Commands

### Building the Project
```bash
dotnet build
```

### Publishing the Application (Self-contained)
```bash
dotnet publish -c Release -r win-x64 --self-contained true
```

### Running the Application
```bash
dotnet run
```

### Building for Specific Configuration
```bash
dotnet build -c Release
```

## Key Features

1. **Port Configuration**: Configures server ports for Legend of Mir game servers
2. **Dynamic Monster Spawning**: Manages monster spawning configurations
3. **Script Optimization**: Optimizes game scripts for better performance

The application automatically detects the game engine type and adjusts configurations accordingly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LiuLei-jm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LiuLei-jm)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
