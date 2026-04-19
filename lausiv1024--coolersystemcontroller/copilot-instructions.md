## coolersystemcontroller

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Blazor WebAssembly application with ASP.NET Core hosting that serves as an IR (Infrared) remote control web interface for controlling a circulator/fan device.

## Key Commands

### Development
```bash
# Run the application
dotnet run --project CoolerSystemController

# Build the solution
dotnet build

# Clean build artifacts
dotnet clean

# Restore dependencies
dotnet restore
```

### Publishing
```bash
# Publish to the configured network share
dotnet publish -p:PublishProfile=FolderProfile
```

## Architecture

### Project Structure
- **CoolerSystemController/** - ASP.NET Core server project that hosts the Blazor app
  - Serves the WebAssembly client
  - Contains the IR API controller
  - Handles communication with external IR device

- **CoolerSystemController.Client/** - Blazor WebAssembly client project
  - Contains the UI components and pages
  - Communicates with server API for IR commands

### Key Components

**Server-Side:**
- `IRController.cs` - API controller that receives IR commands from the client and forwards them to the physical IR device at `http://192.168.1.22/simplecode`
- `SimpleIRCode.cs` - Data model for IR command transmission

**Client-Side:**
- `Circulator.razor` - Main control interface for the circulator with power and speed controls
- `MainLayout.razor` - Application layout wrapper
- `NavMenu.razor` - Navigation component

### Communication Flow
1. User interacts with Blazor UI (Circulator.razor)
2. Client sends POST request to `/api/ir` with SimpleIRCode payload
3. Server IRController forwards the command to external IR device
4. IR device executes the command on the physical circulator

### Development URLs
- HTTP: http://localhost:5187
- HTTPS: https://localhost:7011

## Important Notes

- The application targets .NET 8.0
- No test projects are included in the solution
- Publishing is configured to deploy to `\\LATTESERVER\KougiSiryou\Dev\IRController`
- The external IR device endpoint (`http://192.168.1.22/simplecode`) is hardcoded in the IRController

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lausiv1024) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
