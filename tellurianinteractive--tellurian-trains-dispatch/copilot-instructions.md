## tellurian-trains-dispatch

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a .NET library for dispatching trains between stations on a model railway.
- The **Broker** is the central singleton component holding state of each train's calls at specific stations.
- A **Dispatcher** presents state of arriving and departing trains. The **StationDispatcher** presents state for a named station.

## Domain Model Concepts

### Core Entities
- **Train** - Identified by Company and Identity (e.g., "SJ IC 123"). Has a collection of TrainSection sections.
- **Station** - A named location with a signature. Can be manned or unmanned.
- **StationTrack** - A specific track/platform at a station.
- **DispatchStretch** - The track section between two adjacent stations. Has capacity (single/double track) and optional block signals.
- **TrainSection** - A train's scheduled movement over a DispatchStretch, with departure and arrival calls.
- **TrainStationCall** - A scheduled stop: train, station, track, and call times.
- **BlockSignal** - Intermediate signal point on a DispatchStretch, enabling finer capacity control.

### State Machines
- **TrainState**: Planned → Manned → Running → [Completed | Canceled | Aborted]
- **DispatchState**: None → Requested → [Accepted | Rejected] → Departed → [PassBlockSignal*] → Arrived (or Accepted → Revoked)
- **BlockPassageState**: Expected → Passed (or Canceled)

### Block Signal Handling
A DispatchStretch can be divided into blocks by intermediate block signals:
- **N block signals** create **N+1 blocks** (segments)
- Each block can hold only **one train** at a time
- Trains must pass block signals **in sequence**
- A train's current block index = count of passed BlockSignalPassages
- Arrival is only allowed after **all block signals have been passed**

Block occupancy determines capacity:
- When a train departs, it occupies Block 0 (station to first signal)
- When a dispatcher marks the train as passed a signal, it moves to the next block
- The previous block is then free for the next train

The dispatcher who controls a block signal (`BlockSignal.ControlledBy`) is presented with the action to mark the passage.

## Architecture Patterns

- **Extension Methods** - Behavior is organized into extension method classes for separation of concerns.
- **Option<T>** - Generic error handling for operations that can fail with messages.
- **Dependency Injection Ready** - Core interfaces (IBrokerConfiguration, IBrokerStateProvider, ITimeProvider) allow flexible integration.
- **Thread-Safe ID Generation** - Auto-incrementing IDs use Interlocked for safety.

## Component Relationships

Broker (singleton) manages:
- Collection of DispatchStretch (track sections)
- Collection of StationDispatcher (one per station)
- TrainSection instances representing active dispatching

StationDispatcher exposes arrivals and departures for its station by querying the Broker.

## .NET Version
This project targets .NET 10.0 and the .NET 10 SDK. This is only defined globally in Directory.Build.props.
Test projects should use MSTest.Sdk, the Microsoft Testing Platform and NOT VS Test.

## Documentation
There is a README in the project, outlining the details and intended use.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tellurianinteractive) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
