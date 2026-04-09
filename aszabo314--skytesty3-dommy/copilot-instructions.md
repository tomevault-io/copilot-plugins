## skytesty3-dommy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The project of interest is called `Testy3`. In this project, we use Aardvark.Dom to build an interactive data visualization tool. The tool provides basic modeling and simulation capabilities for solar radiation and photovoltaic panels on buildings based on BIM-Specifications. The photovoltaic panels' performance can be analyzed using novel visualization techniques, and the tool helps with planning solar panel solutions. 

Aardvark.Dom is an F# library for building interactive web applications with real-time 3D rendering capabilities. It combines HTML DOM manipulation with high-performance 3D scene graphs using the Aardvark rendering platform. The architecture follows an Elm-like pattern with adaptive (incremental) computation using FSharp.Data.Adaptive.

## Build System

This project uses **Paket** for dependency management and **.NET 8.0**. Build commands:

- **Build entire solution**: `dotnet build Aardvark.Dom.sln`
- **Build specific project**: `dotnet build src/<ProjectName>/<ProjectName>.fsproj`
- **Restore dependencies**: `dotnet tool restore` then `dotnet paket restore`
- **Run Demo app**: `dotnet run --project src/Demo/Demo.fsproj`
- **Run Testy3 app**: `dotnet run --project src/Testy3/Testy3.fsproj`

The solution outputs to `bin\$(Configuration)\` (consolidated output directory).

## Key Architecture

### Core Libraries (Dependency Order)

1. **Aardvark.Dom.Core**: Foundation types including Worker API
2. **Aardvark.Dom**: Core DOM abstractions and 3D scene graph integration
   - `UI/DomNode.fs`: DomNode types (Element, VoidElement, Text, RenderControl)
   - `UI/Attribute.fs` & `AttributeMap.fs`: HTML attribute system
   - `SceneGraph/`: 3D scene graph nodes, picking (BVH, Intersectable), event handling
   - `Frontend.fs`: DOM rendering and diffing
   - `Updater.fs`: Update loop management
3. **Aardvark.Dom.Elm**: Elm-architecture pattern (App type, Env)
   - App.fs defines `App<'model, 'amodel, 'msg>` with init/update/view
   - `Env<'msg>` provides runtime, message emission, modal UI, worker management
4. **Aardvark.Dom.Remote**: Abstract remote HTML backend for client-server sync
5. **Aardvark.Dom.Server**: Server-side rendering with WebSocket sync, JPEG/SharedMemory transfer
6. **Aardvark.Dom.Giraffe**: Giraffe web framework integration (Server.fs, Route.fs)
7. **Aardvark.Dom.Bootstrap**: Bootstrap UI components (Modal, Dropdown, Accordion, etc.)
8. **Aardvark.Dom.Utilities**: Camera controllers (OrbitController, FreeFlyController)

### Application Pattern

Apps use an Elm-like architecture:
- **Model**: Application state (often using Adaptify for adaptive models with .g.fs files)
- **Update**: `Env<'msg> -> 'model -> 'msg -> 'model` - message handler
- **View**: `Env<'msg> -> 'amodel -> DomNode` - renders adaptive model to DOM

The `DomNode` type represents the virtual DOM, supporting:
- Standard HTML elements with adaptive attributes
- 3D `RenderControl` nodes that integrate Aardvark.SceneGraph
- Event handlers for mouse, touch, keyboard, and custom 3D scene events

### 3D Scene Integration

RenderControl nodes embed 3D scenes with:
- Adaptive scene graphs (`ISceneNode`)
- Picking support (pixel-based and geometric/IIntersectable)
- Scene events (pointer events with 3D hit information, pick rays)
- View/projection matrices for camera control

## Development Notes

- **Adaptify**: Many model types use Adaptify code generation (`<LocalAdaptify>true</LocalAdaptify>`). This creates `.g.fs` files with adaptive wrapper types for efficient incremental updates.
- **paket.dependencies**: Central package management - update here, then run `dotnet paket install`
- **JavaScript integration**: `aardvark-dom.js` files in Remote/Demo provide client-side DOM synchronization
- **Server startup**: Demo and Testy3 apps use Giraffe + WebSockets, often with Aardium for desktop embedding
- **Port configuration**: Demo typically runs on port 53675 (see launchSettings.json files)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aszabo314)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aszabo314)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
