## rh126-quarkus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vegas Trivia is an interactive trivia game application built with Quarkus, designed for Red Hat's 2026 sales kickoff. It demonstrates Quarkus full-stack capabilities through a simple, engaging trivia game with real-time leaderboard and integrated product messaging.

**Key Product Goals:**
- Demonstrate Quarkus capabilities through an interactive experience
- Enable field sales teams to engage Java development teams in modernization conversations
- Provide hands-on demonstration of modern Java development with Quarkus

## Architecture

### Full-Stack Quarkus Application
This is a single Quarkus application serving both backend REST APIs and frontend SPA:
- Backend: Quarkus RESTEasy Reactive with Java 17+
- Frontend: SPA using Quarkus Web Bundler (JavaScript/TypeScript in `src/main/resources/web/`)
- Storage: In-memory HashMap-based storage (no external database for MVP)
- Authentication: Simple username-based (no complex auth, stored in browser localStorage)

**Web Bundler Integration:**
- The `io.quarkiverse.web-bundler:quarkus-web-bundler` extension bundles frontend assets
- Uses Deno and esbuild (no Node.js required)
- Frontend files go in `src/main/resources/web/`
- Supports JavaScript, TypeScript, CSS, SCSS, and TailwindCSS
- Provides live reload for both backend and frontend in dev mode

### Data Model
Three primary data structures (in-memory HashMaps):
1. **Question Bank** - Questions with multiple-choice answers and correct answer tracking
2. **User Responses** - Maps usernames to their answer history
3. **Leaderboard** - Maps usernames to scores based on correct answers

### API Structure
- `GET /api/questions` - Retrieve trivia questions with randomized informational messages
- `POST /api/answers` - Submit answers and validate against correct answers
- `GET /api/leaderboard` - Retrieve current rankings sorted by score

## Development Commands

### Project Setup
```bash
# Create new Quarkus project (if not exists)
quarkus create app com.redhat:vegas-trivia --extension='resteasy-reactive-jackson,web-bundler'

# Or with Maven
mvn io.quarkus:quarkus-maven-plugin:create \
  -DprojectGroupId=com.redhat \
  -DprojectArtifactId=vegas-trivia \
  -Dextensions='resteasy-reactive-jackson,web-bundler'
```

### Build and Run
```bash
# Development mode with live reload (both backend and frontend)
./mvnw quarkus:dev
# or
quarkus dev

# Package application
./mvnw package

# Run packaged application
java -jar target/quarkus-app/quarkus-run.jar
```

### Testing
```bash
# Run all tests
./mvnw test

# Run single test class
./mvnw test -Dtest=QuestionResourceTest

# Run tests in continuous mode during development
./mvnw quarkus:test
```

### Build Optimization
```bash
# Create native executable (requires GraalVM)
./mvnw package -Pnative

# Create container image
./mvnw package -Dquarkus.container-image.build=true
```

## Implementation Roadmap

The product roadmap (`agent-os/product/roadmap.md`) defines feature priorities. Key implementation sequence:
1. Data models and in-memory storage (HashMaps)
2. Backend REST APIs for questions, answers, leaderboard
3. Frontend SPA with Web Bundler setup
4. UI components for login, questions, answers, leaderboard
5. Integration testing and polish

## Key Constraints and Design Decisions

### Event-Optimized Design
- **In-memory storage only** - Data persists only during application runtime (acceptable for sales kickoff event context)
- **No complex authentication** - Username-based login with browser storage is sufficient
- **Lightweight architecture** - Fast startup, low memory footprint for demonstration purposes

### Messaging Integration
The application includes randomized informational messages about Quarkus that display alongside trivia questions. These messages are integrated into the question API response and rotate randomly to naturally expose users to key value propositions.

### Full-Stack Architecture
Backend and frontend are served from a single Quarkus application. The Web Bundler extension handles asset compilation and bundling, eliminating the need for separate Node.js tooling or build processes.

## Reference Documentation

Product planning documents in `agent-os/product/`:
- `mission.md` - User personas, problem statement, key features
- `tech-stack.md` - Complete technology stack and architecture decisions
- `roadmap.md` - Feature implementation sequence and priorities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeremyrdavis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
