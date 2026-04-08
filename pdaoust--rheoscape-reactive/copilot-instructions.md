## rheoscape-reactive

> When creating a plan in Claude Code:

# Claude Code Workflow Guide

## Session Management

### Planning Sessions

When creating a plan in Claude Code:

1. **Initial Planning**: Create a detailed plan and checklist for review
2. **Review Gate**: Wait for human approval/modifications before proceeding
3. **Save Plan and Checklist**: Once approved, save to a session file called `./CLAUDE_SESSION_<descriptive_name>.md`
5. **Initial Commit**: Commit both files with message: `"created plan and checklist for <summary>"`

### Session Recovery

The session file must contain sufficient context to:
- Resume work after hardware crashes or session interruptions
- Understand the goals and progress of the session
- Continue from the last completed checkpoint

### Progress Tracking

- Update session file as plan and checklist items are completed
- Update session file whenever the plan and checklist are revised
- Each completed checklist item triggers a commit containing:
  - The work completed for that item
  - Updated session file showing new status
  - Commit message: `"<commit type>: <descriptive message for checklist item>"`. The commit type uses [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) to indicate the type of change.

### Commit Granularity

Each commit should represent a logically cohesive unit of work corresponding to a checklist item.

---

## Project Context

### Project Purpose

Rheoscape is a C++ pattern and standard library for embedded programming, linking data sources and sinks together in pipelines using operators that transform the streams in useful ways. It's closely related to functional reactive programming. The difference between Rheoscape and other FRP libraries is that Rheoscape supports both observable streams and iterable streams with a single interface, allowing developers to use the same operators for both kinds of streams. It also allows developers and consumers of sources/sinks to have more choice about whether to implement push-based or pull-based functional reactive programming, or a hybrid of the two.

Key concepts:

* A **source function** defines a stream. It receives one input argument, a **push function**, a callback that receives values from the source as they are emitted. It returns a **pull function**, which (if the source function supports it) causes the source function to emit the next available value in the stream to the bound push function.
* A **sink** is a vague concept -- a sink could simply be a push function that manipulates state that it has access to in lexical context, or it could be a **sink function**, which takes a source function, binds an internal push function to it, and stores/uses its returned pull function.
* If a source function pushes its values without being asked (e.g., triggered by an interrupt), it's called a **push source**. If it only pushes values when its pull function is called, it's called a **pull source**. In practice, because Rheoscape has no understanding of concurrency, there are very few (if any) true push sources except ones that are triggered manually in a program main loop. Emitted values should be assumed to be able to pass from source down to final sink without interruption. This creates a model of execution that's easier to reason about.
* Each act of binding a push function to a source function generates a new independent stream; therefore a source function can be thought of as a stream factory. Two bindings to the same source function may incidentally receive the same values when their respective pull functions are called, but calling pull function #1 will not cause the source function to emit a value to push function #2.
* One common idiom is a **source function factory**, a function that takes one or more source functions and other parameters and returns a new source function. Source functions usually show up as **operators**, which take a source and produce a source that yields a transformed stream (e.g., a `filter` operator takes a source function and a predicate callable, and returns a new source function whose stream only contains values that match the predicate).
* Another common idiom is a **pipe function factory**, which is like a source function factory with partial application -- it usually corresponds to a source function factory but its 'main' upstream source function can't be passed as a parameter -- instead, all the other parameters are bound to a returned **pipe function** which accepts the main source function as its only argument. A pipe function is actually a lot like a source function factory except it has a consistent interface -- one source function in, one source function out. This lets you pipe a single source function with a lot of pipe function factories in a very ergonomic way, using an operator overload for the `|` operator.
* If a source function wants to communicate that it may yield errors, it should use `rheoscape::Fallible` as the type for its stream rather than throwing exceptions.
* If a source function wants to communicate that it will eventually run out of values, it may use `rheoscape::Endable` as the type for its stream.

### Target Audience

Embedded developers who want to use a roughly flow-based programming approach to constructing streams from data sources (sensors, input events, iterables, etc) to data sinks (LCD displays, GPIOs, actuators, HTTP requests, etc) in a fairly rapid and ergonomic way.

### Key Constraints

- C++ 20 is a hard requirement, because it gives affordances that make code much more readable.
- As a framework for embedded systems, low memory consumption and high performance are priorities.
- Developer ergonomics is a high priority -- the experience of developers consuming Rheoscape should be prioritised first, followed by developers of sources, sinks, and operators.
- I currently only have the resources to test on ESP32, RP2040, and Atmel AVR hardware.

---

## Architecture Overview

### System Structure

The pattern, idioms, and type traits / concepts are defined in one file. That's the core of Rheoscape. There's also a standard library that should meet common IoT needs:

- **Sources**: Found in `src/sources/`, sources for iterable data types, clocks, GPIOs, and Arduino-compatible sensors.
- **Sinks**: Found in `src/sinks`, sinks for GPIOs, actuators, display, logging, etc.
- **Operators**: Found in `src/operators`, a bunch of common operators.
- **Types**: Helper types.

### Data Flow

Sources emit streams of data through pipes that transform the data, then eventually arrive at final sinks to perform some action. Requests for data bubble up through the pipes, reach the source, which emits data to listening pull_fn callbacks all the way back down the pipe to the sink. Each intermediate step in a pipe is both a source and a sink. Not all sources need to be pulled (and indeed, some can't).

### External Dependencies

* Many sources depend heavily on the `au` units library from Aurora Open Source to give SI units to their values (e.g., temperature sensors use `au::QuantityType<DegreesC, float>`).
* Many sources and sinks also make heavy use of the Arduino framework and various Arduino-based libraries for individual hardware components.
* The whole project is based on PlatformIO, which is not needed for third-party development but is highly recommended for developing this library, as it's useful for pulling in dependencies and running tests.
* Graphics sinks use LVGL 9.
* HomeAssistant and MQTT sinks/sources (not currently usable) use external dependencies.

---

## Development Guidelines

### Code Organization

- `src`: the library
  - `src/home_assistant`: sources and sinks for integration with Home Assistant (not currently usable)
  - `src/mqtt`: sources and sinks for exposing the device to MQTT servers (not currently usable)
  - `src/operators`: a standard library of functional operators
  - `src/sinks`: sinks for IoT and generic data stuff
  - `src/sources`: sources for IoT and generic data stuff
  - `src/types`: useful helper types
  - `src/ui`: sinks for LVGL-driven displays
  - `src/types/core_types.hpp`: the most important file, which defines Rheoscape's core types and idioms
  - `src/**/everything.hpp`:
- `test`: unit and integration tests
- Naming conventions: snake_case for functions and primitive types, PascalCase for structs and classes. Private struct and class members start with an underscore.
- Comments should:
  - precede the code they're commenting -- no inline comments
  - use semantic line breaks at clause or phrase boundaries
  - be ideally 120 characters per line or less, including whitespace preceding comment symbol
  - use full sentences with proper terminal punctuation.
- Every source, sink, or operator should live in a separate file. Extra types that are only used by the source, sink, or operator and their downstream sinks may be defined in that file; otherwise, if a type may be used by multiple components, it should be put into `src/types`.
- This is a header-only library; all components should be in `.hpp` files.
- Guard against multiple import with `#pragma once`.

### Quality Standards

- Testing requirements: Every source, sink, and operator should have a corresponding set of unit tests in `test/` that exercises its use with trivial sources and sinks and explores edge cases and possible race conditions with parallel pipes. Sources and sinks that use embedded hardware should have a guard macro that prevents the test from running on the wrong hardware (check `platformio.ini` for currently supported environments and the `PLATFORM_*` flags that they set)
- Documentation requirements: Every source, sink, and operator should have file-level
- Code review expectations: [what should reviewers focus on?]

### Common Patterns

- Operators typically come in two flavours: a source function factory that receives a source and any required other parameters (which themselves may be source functions) and returns a pull function, and a pipe function factory that receives the same parameters minus the initial source and returns a pipe function that receives the initial source and returns a transformed source.
- For operators where it's semantically meaningful to take multiple like-kinded source functions, the operator function should have variadic generic types -- that is, it should take a list or pack of source functions of arbitrary types. If the operator requires a mapper, predicate, combiner, or other sort of predicate, that parameter should come before the variadic parameters.
- Because Rheoscape relies heavily on functions, inner functions that are returned from outer functions (such as source functions and pull functions) should be named callables rather than anonymous lambdas. This helps make the call stack trace more readable in case of errors.
- Operators should not handle exceptions directly; instead, they should bubble up to the original call site that constructed the pipe.
- Sinks may throw exceptions.

### Anti-Patterns

- Sources may throw exceptions, but it's preferred for them to emit a stream of `rheoscape::Fallible` values instead.
- Components should not assume concurrency.
- Components should be as ignorant as possible about ownership semantics of inputs, outputs, and streamed values, deferring such decisions to their upstream sources and downstream sinks wherever possible.

---

## Build and Development

### Setup Instructions

TODO

### Build System

Use PlatformIO for everything; it'll make your life easier!

### Development Workflow

TODO

### Environment Variables

TODO

---

## Testing Strategy

### Test Organization

Tests live in `test/` and are organised into folders for operators, sources, and reusable types. Each file should cover an individual component and use the Unity testing library.

### Running Tests

Use Platformio's built-in test command; you can find it at `~/.platformio/penv/bin/pio`. Always run tests locally using `-e dev_machine`.

### Test Philosophy

TODO

---

## Known Issues and Gotchas

### Current Limitations

TODO
I know concurrency of all sorts isn't supported, which makes the execution model really easy to reason about and build for. I'm not sure what it would take to support concurrency (because embedded devices use interrupts, and increasingly have multiple cores), but I suspect it'll make the framework really unpleasant to support.

### Common Pitfalls

Writing a source function factory or pipe function factory involves writing multiple layers of inner functions whose control flow are really hard to follow sometimes. It's really easy to write an operator that doesn't do exactly what you think it would.

### Technical Debt

TODO

---

## Communication Preferences

### Question Handling

- Ask clarifying questions when requirements are ambiguous
- Propose solutions with tradeoffs rather than asking what to do
- Flag potentially breaking changes before implementing
- Look for ways to improve the elegance and ergonomics of the codebase by simplifying code, but always favour the ergonomics of the end-user developer
- Do not ask me to confirm whether to run non-destructive tools during discovery such as `git show`, `find`, and `grep` -- just run them

### Error Reporting

- When refactoring, look for existing code with:
  - ambiguous or problematic ownership semantics for streamed values or received/returned functions
  - race conditions when splitting or merging streams
  - other things that don't look right
  - use of old C++ idioms that can be updated with modern C++20 idioms

### Decision Making
- If you're refactoring multiple components and each set of changes doesn't differ much from the previous component's set of changes, just apply them without asking.
- Global renames or replacement of deprecated types with new types should also be applied without asking.
- Present challenging architectural problems as a series of options; don't assume I know much about C++; assume that I have a lot to learn about improving the quality and performance of mhy codebase.

---

## Resources

### Documentation
- Documentation lives inline with each file. The main documentation is found in `src/types/core_types.hpp`.

### Related Projects
- Rheoscape draws heavy inspiration from [Callbag](https://github.com/callbag/callbag), but ditches the complex initial handshake and the error callbacks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pdaoust)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pdaoust)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
