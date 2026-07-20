## xtream-codec

> - **JDK**: 21 required

# AGENTS.md - xtream-codec

## Build

- **JDK**: 21 required
- **Build tool**: Gradle (Kotlin DSL) via `./gradlew`
- **Run single test**: `./gradlew :module:test --tests "FullyQualifiedTestName"`
- **Build with checkstyle**: `./gradlew clean build -P xtream.backend.build.checkstyle.enabled=true`
- **Skip slow checks locally**: Uses properties in `gradle.properties`:
    - `xtream.backend.build.checkstyle.enabled=false`
    - `xtream.backend.build.errorprone.enabled=false`

## Module Structure

```
xtream-codec-core/          # Core codec (annotation-driven)
xtream-codec-server-reactive/  # Async non-blocking TCP/UDP server
ext/jt/jt-808-server-spring-boot-starter-reactive/   # JT/T 808 impl
ext/jt/jt-808-server-dashboard-spring-boot-starter-reactive/  # JT/T 808 Dashboard
ext/jt/jt-1078-server-spring-boot-starter-reactive/  # JT/T 1078 impl
quick-start/              # Runnable examples
debug/                   # Debug modules (ignore)
```

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Code Conventions

**These rules must be followed in ALL generated code.**

### 5.1 @since Tag Version

When adding `@since` to JavaDoc, the value MUST match `projectVersion` in `gradle.properties`.

- Current version: `0.6.0` (`projectVersion=0.6.0` in `gradle.properties`)
- New APIs added now → `@since 0.6.0`
- Do NOT hardcode outdated versions; always check `gradle.properties` first.

```java
// Correct (new code):
/**
 * @since 0.6.0
 */
default boolean isDerived() { return false; }

// Wrong (version doesn't match gradle.properties):
/**
 * @since 0.4.0
 */
default boolean isDerived() { return false; }
```

### 5.2 @Nullable Placement (Jspecify)

`@Nullable` is a **type-use annotation**. It MUST be placed immediately before the type it modifies, NOT on a separate line before the method declaration.

```java
// Correct — @Nullable before the return type:
public @Nullable String getDisplayName() { ... }
default @Nullable Object getProperty(Object instance) { ... }
public static @Nullable String getVariable(String name) { ... }

// Wrong — @Nullable on its own line before default/modifier:
// @Nullable         ← wrong
// default Object getProperty(...) { ... }

// Wrong — @Nullable separated from the return type by a modifier:
// @Nullable private String name;              ← wrong
// private @Nullable String name;              ← correct (no modifier between @Nullable and the type)
```

Rationale: Per [Jspecify 1.0](https://jspecify.dev/), type-use annotations should be adjacent to the annotated type to avoid ambiguity about what they modify.

### 5.3 Comment Language

All code comments (including JavaDoc, inline comments, TODO, FIXME, etc.) MUST be written in **Simplified Chinese** unless the comment targets an international audience (e.g., SPI interface docs meant for external contributors).

```java
// Correct (简体中文):
// 将状态码转换为业务枚举
@Nullable
StatusEnum resolveStatusCode(int code);

// Wrong (English comments in Chinese project):
// Convert status code to business enum
@Nullable
StatusEnum resolveStatusCode(int code);
```

---
> Source: [hylexus/xtream-codec](https://github.com/hylexus/xtream-codec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
