## medium-download

> You are assisting with a Node.js Medium blog scraper that downloads posts as markdown files. This project uses **Acceptance Test Driven Development (ATDD)** with Jest and functional programming patterns.

# Claude Code Assistant Instructions - Medium Blog Scraper

## 🤖 AI Context & Project Overview

You are assisting with a Node.js Medium blog scraper that downloads posts as markdown files. This project uses **Acceptance Test Driven Development (ATDD)** with Jest and functional programming patterns.

### Key Project Files

- `features/medium-scraper.feature` - Gherkin scenarios defining all requirements
- `test/acceptance/` - Jest BDD tests translating scenarios
- `SETUP.md` - Initial project setup instructions

## 🎯 Primary Objectives

1. **Follow ATDD workflow strictly** - Read scenarios → Write tests → Implement → Validate
2. **Enforce quality gates** - Run `npm test` and `npm run quality` after EVERY change
3. **Use functional programming** - No classes, only arrow functions and pure functions
4. **Maintain BDD test structure** - Tests must mirror Given/When/Then scenarios

## ⚡ Quick Command Reference

```bash
# Essential commands - Run after EVERY change
npm test                    # Run all tests
npm run quality            # Check code quality

# Development commands
npm test -- --watch        # Watch mode for TDD
npm test -- --testNamePattern="<pattern>"  # Run specific test
npm test -- --testPathPattern="<path>"     # Run tests in path
```

## 🚨 CRITICAL RULES - NO EXCEPTIONS

### After Every Code Change

1. ✅ Run `npm test` - ALL tests must pass
2. ✅ Run `npm run quality` - NO linting/formatting errors
3. ✅ Fix any failures before proceeding
4. ✅ Never skip for "small changes"

### Code Style Mandates

- **ES Modules only** - Use import/export syntax
- **Arrow functions only** - No function declarations
- **No classes** - Use factory functions and closures
- **No semicolons** - Enforced by Prettier
- **Functional patterns** - Pure functions, immutability
- **80 char line limit** - Keep code readable

## 📋 Implementation Checklist

When implementing each feature:

- [ ] Read the scenario in `features/medium-scraper.feature`
- [ ] Create Jest BDD test in `test/acceptance/`
- [ ] Run test (should fail initially)
- [ ] Implement minimal code to pass test
- [ ] Run `npm test` - verify pass
- [ ] Run `npm run quality` - verify clean
- [ ] Refactor if needed (keeping tests green)
- [ ] Move to next scenario

## 🏗️ Project Structure

```
project/
├── features/
│   └── medium-scraper.feature    # Requirements as Gherkin scenarios
├── test/
│   └── acceptance/               # BDD-style Jest tests
│       ├── auth.test.js         # Authentication tests
│       ├── scraping.test.js     # Scraping tests
│       └── storage.test.js      # Storage tests
├── src/
│   ├── auth/                    # OAuth implementation
│   ├── scraper/                 # Scraping logic
│   └── storage/                 # File management
└── CLAUDE.md                     # This file
```

## 💡 Test Pattern Template

Use this BDD pattern for all acceptance tests:

```javascript
// test/acceptance/<feature>.test.js
describe('Feature: <Feature Name>', () => {
  describe('Scenario: <Scenario Name>', () => {
    let result

    beforeEach(() => {
      // Given setup
    })

    describe('When <action>', () => {
      beforeEach(async () => {
        // Action execution
        result = await performAction()
      })

      it('Then <expected outcome>', () => {
        expect(result).toBe(expectedValue)
      })

      it('And <additional outcome>', () => {
        expect(sideEffect).toHaveBeenCalled()
      })
    })
  })
})
```

## 🔄 Development Workflow

### Phase 1: Setup & Authentication

1. Review authentication scenarios in feature file
2. Implement Google OAuth flow with tests
3. Validate token storage and refresh

### Phase 2: Profile Discovery

1. Test Medium profile URL parsing
2. Implement post listing retrieval
3. Handle pagination for all posts

### Phase 3: Content Scraping

1. Test markdown conversion
2. Implement image downloading
3. Update image references to local paths

### Phase 4: Storage & Organization

1. Test directory structure creation
2. Implement file naming strategies
3. Handle duplicates and updates

## 🛠️ Common Tasks

### Adding a New Feature

```bash
# 1. Read the scenario
cat features/medium-scraper.feature | grep -A 10 "Scenario: Your Feature"

# 2. Create test file
touch test/acceptance/your-feature.test.js

# 3. Run test in watch mode
npm test -- --watch --testPathPattern="your-feature"

# 4. Implement until green
# 5. Check quality
npm run quality
```

### Debugging Failed Tests

```bash
# Run with verbose output
npm test -- --verbose

# Run single test with debugging
node --inspect-brk node_modules/.bin/jest --runInBand
```

## 📝 Current Implementation Status

Track progress here:

- [x] Authentication flow
- [x] Profile discovery
- [x] Post listing
- [x] Content scraping
- [x] Image handling
- [ ] Storage organization
- [ ] Duplicate detection
- [ ] Incremental updates

## 🔍 Key Implementation Details

### Authentication

- Use Google OAuth 2.0 for Medium SSO
- Store tokens securely (consider keychain/credential manager)
- Implement automatic token refresh

### Scraping Strategy

- Parse Medium's HTML structure (may change)
- Extract clean markdown from post content
- Preserve code blocks and formatting
- Download images with proper naming

### Storage Format

```
output/
├── posts/
│   ├── 2024-01-15-post-title.md
│   └── 2024-02-20-another-post.md
└── images/
    ├── post-title/
    │   ├── image1.jpg
    │   └── image2.png
    └── another-post/
        └── header.jpg
```

## 🚀 Quick Start for Claude Code

When starting a session:

1. **Read this file first** for context
2. **Check implementation status** above
3. **Review feature file** for current scenario
4. **Run tests** to see current state
5. **Continue from last failing test**

## 📌 Remember

- **Quality gates are mandatory** - No exceptions
- **Tests drive development** - Write test first
- **Functional style only** - No OOP patterns
- **Follow the scenarios** - They define success
- **Run quality checks** - After every change

---

_This document is optimized for Claude Code. Always refer to `features/medium-scraper.feature` for authoritative requirements._

---
> Source: [bdfinst/medium-download](https://github.com/bdfinst/medium-download) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
