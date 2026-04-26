## actiondesigner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Unity project called "Action Designer" (AD) - a visual node-based action design system similar to behavior trees. The project implements a flowchart system where Behavior and Condition nodes can be connected to control Unity object behaviors.

## Architecture

### Core Components

- **ActionDesigner.Runtime**: Core runtime system
  - `Action.cs`: Main action graph container with node management and dynamic node creation
  - `BaseNode.cs`: Abstract base class for all nodes with serialization support
  - `BehaviorNode.cs`/`ConditionNode.cs`: Concrete node implementations with IBehavior/ICondition objects
  - `ActionRunner.cs`: Runtime execution system
  - `IBehavior.cs`/`ICondition.cs`: Core interfaces for pluggable behaviors and conditions

- **ActionDesigner.Editor**: Unity Editor tooling
  - `ActionDesignerEditor.cs`: Main editor window using UI Toolkit with custom UI composition
  - `CustomActionView.cs`: Visual graph editor with node manipulation (replaces ActionView.cs)
  - `CustomNodeView.cs`/`SimpleCustomNodeView.cs`: Node visualization components
  - `NodeSearchWindow.cs`: Node creation search interface
  - `UIToolkitNodeInspector.cs`: Property inspector for selected nodes

### Key Systems

1. **Node System**: Type-based dynamic node creation using reflection and SerializeReference
   - Nodes store type/namespace strings and create concrete objects at runtime
   - Uses `Action.GetOperationType()` for cached type resolution
   - `BaseNode.CreateNodeObject()` instantiates IBehavior/ICondition objects

2. **Behavior System**: Pluggable behavior implementations in `Tasks/` folder
   - `DebugLogTask`, `WaitTask`: Basic behaviors
   - `Parallel`, `Sequencer`: Composite behaviors for flow control
   - All behaviors implement `IBehavior` interface with `Start()` and `Update()` methods

3. **Condition System**: Runtime condition evaluation in `Conditions/` folder
   - `KeyPress`, `EndCondition`, `AlwaysTrue`: Example condition implementations
   - All conditions implement `ICondition` interface

4. **Editor Integration**: Custom UI Toolkit implementation
   - `CustomOrthogonalEdge.cs`/`SimpleOrthogonalEdge.cs`: Custom edge rendering
   - Property drawers using SerializeReferenceExtensions for polymorphic object selection

### External Dependencies

- **SerializeReferenceExtensions**: Custom serialization support for polymorphic references
  - Provides `[SubclassSelector]` attribute for editor dropdown selection
  - Located in `Assets/SerializeReferenceExtensions/`
- **InputSystem**: Custom input handling system (referenced from ExternalModule project)

## Development Commands

This is a Unity project using Unity 6000.0.54f1:

- Open project in Unity Editor (Unity 6000.0.54f1 or compatible)
- Build through Unity Editor: File → Build Settings  
- Testing done through Unity Test Runner (Window → General → Test Runner)
- Access Action Designer editor: **Action Designer → Editor...** menu

## Node Creation Workflow

1. **Creating Nodes**: Use the node search window to create new behavior/condition nodes
2. **Type Assignment**: Nodes are created with type/namespace strings, objects instantiated via reflection
3. **Connection System**: Parent-child relationships stored as ID lists in `BaseNode.childrenID`
4. **Runtime Execution**: `ActionRunner` traverses the node graph and executes behaviors/conditions

## Key Features Implemented

- Visual node-based action designer with drag-and-drop interface
- Runtime action execution system with Start/Update lifecycle
- Condition evaluation system for flow control
- Parallel and sequential task execution through composite behaviors
- Node search and creation system with type-based instantiation
- Title editing and precise node positioning
- Custom UI Toolkit editor with orthogonal edge rendering

## Recent Development Focus

Based on recent commits and file changes:
- Behavior terminology (renamed from "Motion" to "Behavior")
- Node precise positioning functionality
- Title editing without breaking child connections
- Parallel sequencer development
- Custom editor views and edge rendering components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/parkJuicy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
